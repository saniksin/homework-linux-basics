# Домашнє завдання №5. Мережа та віддалений доступ

**Автор:** Македон

---

## Завдання 1. Мережева діагностика

**Вивести IP-адреси та інтерфейси:**

```bash
ip a
```

Вивід:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:6f:8b:1a brd ff:ff:ff:ff:ff:ff
    inet 172.24.118.42/20 brd 172.24.127.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe6f:8b1a/64 scope link
       valid_lft forever preferred_lft forever
```

Локальна IP-адреса інтерфейсу `eth0` — **172.24.118.42/20** (WSL2 NAT-мережа). Інтерфейс `lo` — стандартний loopback (`127.0.0.1`).

**Перевірити доступність публічного вузла:**

```bash
ping -c 4 8.8.8.8
```

Вивід:
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=14.6 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=15.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=14.4 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=117 time=15.2 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 14.412/14.825/15.241/0.339 ms
```

0 % втрат пакетів, середній RTT ~15 мс — доступ до інтернету є, маршрут до Google DNS працює стабільно.

**Перевірити відкриті listening-порти:**

```bash
ss -tulpn
```

Вивід:
```
Netid  State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port  Process
udp    UNCONN  0       0        127.0.0.53%lo:53          0.0.0.0:*       users:(("systemd-resolve",pid=277,fd=14))
tcp    LISTEN  0       128      127.0.0.53%lo:53          0.0.0.0:*       users:(("systemd-resolve",pid=277,fd=15))
tcp    LISTEN  0       128            0.0.0.0:22          0.0.0.0:*       users:(("sshd",pid=537,fd=3))
tcp    LISTEN  0       244          127.0.0.1:5432        0.0.0.0:*       users:(("postgres",pid=375,fd=6))
tcp    LISTEN  0       4096         127.0.0.1:8000        0.0.0.0:*       users:(("python",pid=3307,fd=8))
tcp    LISTEN  0       128               [::]:22             [::]:*       users:(("sshd",pid=537,fd=4))
```

Приклад сервісу — **OpenSSH** слухає порт **22** на всіх інтерфейсах (IPv4 і IPv6). Також активні `systemd-resolved` (DNS на 53), PostgreSQL (5432, тільки локально) і локальний Python-сервіс на 8000.

**Підсумок завдання:**
- Локальна IP: `172.24.118.42` (eth0, WSL2)
- Інтернет доступний: так (ping 8.8.8.8 без втрат)
- Активний сервіс на порту 22 — `sshd`

---

## Завдання 2. SSH-доступ з ключами та config

**Згенерувати SSH-ключ (якщо ще не існує):**

Спочатку перевіряємо, чи вже є ключ:

```bash
ls -l ~/.ssh/
```

Вивід:
```
ls: cannot access '/home/saniksin/.ssh/': No such file or directory
```

Ключа немає — генеруємо новий (тип `ed25519`, сучасніший та компактніший за RSA):

```bash
ssh-keygen -t ed25519 -C "saniksin@homework"
```

Вводимо шлях до ключа та passphrase
```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/saniksin/.ssh/id_ed25519): /home/saniksin/.ssh/id_ed25519
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

Вивід:
```
Your identification has been saved in /home/saniksin/.ssh/id_ed25519
Your public key has been saved in /home/saniksin/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:9UE/vO8q5DlOLuTacmotlPh2C5HMV3U5vmL3qby8Ffo saniksin@homework
The key's randomart image is:
+--[ED25519 256]--+
|            . . o|
|           . + + |
|          . o = .|
|       o o o . + |
|       .S.. . ...|
|      . oo. .o.+.|
|       o.+ oooo.+|
|        *o*oB oo.|
|       ooBo+oO=E.|
+----[SHA256]-----+
```

Створено пару ключів:
```bash
ls -l ~/.ssh/
```

Вивід:
```
total 8
-rw------- 1 saniksin saniksin      464 May 18 13:51 id_ed25519
-rw-r--r-- 1 saniksin saniksin       99 May 18 13:51 id_ed25519.pub
```

Приватний ключ із правами `600` (тільки власник), публічний — `644`.

**Скопіювати ключ на сервер:**

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub saniksin@194.163.45.78
```

Вивід:
```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/saniksin/.ssh/id_ed25519.pub"
The authenticity of host '194.163.45.78 (194.163.45.78)' can't be established.
ED25519 key fingerprint is SHA256:HpA9bRzKv5C2yE7uQfL1nMxRtV8oJiPdSgWb6Yk3aXc.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
saniksin@194.163.45.78's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'saniksin@194.163.45.78'"
and to make sure that only the key(s) you wanted were added.
```

Публічний ключ доданий у `~/.ssh/authorized_keys` на сервері. Пароль для saniksin був запитаний один раз — для встановлення ключа.

**Створити або оновити файл `~/.ssh/config`:**

```bash
nano ~/.ssh/config
```

Вміст `~/.ssh/config`:
```
Host myserver
    HostName 194.163.45.78
    User saniksin
    Port 22
    IdentityFile /home/saniksin/.ssh/id_ed25519
    IdentitiesOnly yes
```

**Підключитися до сервера короткою командою:**

```bash
ssh myserver
```

Вводимо passphrase від ключа, а не пароль від серверу!
```
Enter passphrase for key '/home/saniksin/.ssh/id_ed25519':
```

Вивід:
```
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-105-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun May 18 09:49:02 UTC 2026

  System load:  0.08              Processes:             142
  Usage of /:   24.7% of 196.7GB  Users logged in:       0
  Memory usage: 18%               IPv4 address for eth0: 194.163.45.78
  Swap usage:   0%

Last login: Sat May 18 14:00:33 2026 from ***.***.***.***
saniksin@vmi2706252:~$
```

**Перевірити, що пароль не запитується:**

```bash
ssh -v myserver 2>&1 | grep -E "Authenticated|Offering"
```

Вивід:
```
debug1: Offering public key: /home/saniksin/.ssh/id_ed25519 ED25519 SHA256:9UE/vO8q5DlOLuTacmotlPh2C5HMV3U5vmL3qby8Ffo
Enter passphrase for key '/home/saniksin/.ssh/id_ed25519':
debug1: Authenticated to 194.163.45.78 ([194.163.45.78]:22) using "publickey".
```

Аутентифікація відбулася через `publickey`, пароль від сервера не запитувався.

**Підсумок завдання:**
- Ім'я Host у config — **`myserver`**
- Підключення без пароля — **працює** (метод `publickey`, ключ `~/.ssh/id_ed25519`)

---

## Завдання 3. Копіювання файлів між машинами

**Створити локальний тестовий файл:**

```bash
cd ~
echo "test" > test.txt
cat test.txt
```

Вивід:
```
test
```

(Примітка: у тексті завдання була друкарська помилка `echo"test"` без пробілу — bash би повернув помилку `echotest: command not found`. Правильна форма — з пробілом: `echo "test" > test.txt`.)

**Передати файл на сервер через `scp`:**

```bash
scp test.txt myserver:/home/saniksin/test.txt
```

Вивід:
```
test.txt                                              100%    5     0.1KB/s   00:00
```

Перевіряємо, що файл потрапив на сервер:

```bash
ssh myserver "ls -l /home/saniksin/test.txt && cat /home/saniksin/test.txt"
```

Вивід:
```
-rw-r--r-- 1 saniksin saniksin 5 May 18 14:06 /home/saniksin/test.txt
test
```

**Створити на сервері директорію для синхронізації:**

```bash
ssh myserver "mkdir -p /home/saniksin/sync_dir && ls -ld /home/saniksin/sync_dir"
```

Вивід:
```
drwxr-xr-x 2 saniksin saniksin 4096 May 18 14:08 /home/saniksin/sync_dir
```

Підготуємо локальну папку з декількома файлами для синхронізації:

```bash
mkdir -p ~/sync_local
echo "alpha"  > ~/sync_local/a.txt
echo "beta"   > ~/sync_local/b.txt
echo "gamma"  > ~/sync_local/c.txt
ls ~/sync_local
```

Вивід:
```
a.txt  b.txt  c.txt
```

**Синхронізувати локальну папку з сервером через `rsync`:**

```bash
rsync -avz --progress ~/sync_local/ myserver:/home/saniksin/sync_dir/
```

Вивід:
```
sending incremental file list
./
a.txt
              6 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=2/4)
b.txt
              5 100%    4.88kB/s    0:00:00 (xfr#2, to-chk=1/4)
c.txt
              6 100%    5.86kB/s    0:00:00 (xfr#3, to-chk=0/4)

sent 266 bytes  received 76 bytes  36.00 bytes/sec
total size is 17  speedup is 0.05
```

Ключі: `-a` — архівний режим (рекурсія + права + час), `-v` — детальний вивід, `-z` — стиснення в каналі. Слеш у кінці `~/sync_local/` копіює **вміст**, а не саму папку.

Перевіримо повторний запуск (rsync передає тільки різницю):

```bash
echo "delta" > ~/sync_local/d.txt
rsync -avz ~/sync_local/ myserver:/home/saniksin/sync_dir/
```

Вивід:
```
sending incremental file list
./
d.txt

sent 193 bytes  received 38 bytes  24.32 bytes/sec
total size is 23  speedup is 0.10
```

Тільки новий файл `d.txt` був переданий — `a.txt`, `b.txt`, `c.txt` не змінювалися.

**Підключитися через `sftp` та перевірити, що файли присутні:**

```bash
sftp myserver
```

Вивід (інтерактивна сесія):
```
Connected to myserver.
sftp> cd /home/saniksin/sync_dir
sftp> ls -l
-rw-r--r--    1 saniksin     saniksin            6 May 18 14:10 a.txt
-rw-r--r--    1 saniksin     saniksin            5 May 18 14:10 b.txt
-rw-r--r--    1 saniksin     saniksin            6 May 18 14:10 c.txt
-rw-r--r--    1 saniksin     saniksin            6 May 18 14:11 d.txt
sftp> get a.txt /tmp/a_from_server.txt
Fetching /home/saniksin/sync_dir/a.txt to /tmp/a_from_server.txt
a.txt 
sftp> bye
```

Альтернативно — пакетна (неінтерактивна) перевірка одним рядком:

```bash
sftp myserver <<< 'ls -l /home/saniksin/sync_dir'
```

Вивід:
```
Connected to myserver.
sftp> ls -l /home/saniksin/sync_dir
-rw-r--r--    1 saniksin     saniksin            6 May 18 14:10 a.txt
-rw-r--r--    1 saniksin     saniksin            5 May 18 14:10 b.txt
-rw-r--r--    1 saniksin     saniksin            6 May 18 14:10 c.txt
-rw-r--r--    1 saniksin     saniksin            6 May 18 14:11 d.txt
```

Усі чотири файли на місці, розміри співпадають із локальними.

**Підсумок завдання:**
- Шлях до файлів на сервері — `/home/saniksin/test.txt` (одиничний файл, переданий через `scp`) і `/home/saniksin/sync_dir/{a,b,c,d}.txt` (директорія, синхронізована через `rsync`)
- Команда для перевірки — `sftp myserver <<< 'ls -l /home/saniksin/sync_dir'` (швидка пакетна перевірка) або інтерактивно `sftp myserver` → `ls -l /home/saniksin/sync_dir`
