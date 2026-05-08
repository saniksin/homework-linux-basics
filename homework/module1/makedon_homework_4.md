# Домашнє завдання №4. Пакети, сервіси та журнали

## Завдання 1. Менеджери пакетів

**Оновити список пакетів у системі:**

```bash
sudo apt update
```

Вивід:
```
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Get:2 http://archive.ubuntu.com/ubuntu jammy-updates InRelease [128 kB]
Get:3 http://archive.ubuntu.com/ubuntu jammy-security InRelease [129 kB]
Get:4 http://archive.ubuntu.com/ubuntu jammy-backports InRelease [127 kB]
Fetched 384 kB in 1s (412 kB/s)
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
All packages are up to date.
```

**Встановити утиліту `tree`:**

```bash
sudo apt install -y tree
```

Вивід:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  tree
0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
Need to get 47.9 kB of archives.
After this operation, 116 kB of additional disk space will be used.
Get:1 http://archive.ubuntu.com/ubuntu jammy/universe amd64 tree amd64 2.0.2-1 [47.9 kB]
Fetched 47.9 kB in 0s (180 kB/s)
Selecting previously unselected package tree.
(Reading database ... 167328 files and directories currently installed.)
Preparing to unpack .../tree_2.0.2-1_amd64.deb ...
Unpacking tree (2.0.2-1) ...
Setting up tree (2.0.2-1) ...
Processing triggers for man-db (2.10.2-1) ...
```

**Перевірити, що пакет встановлено, та вивести його версію:**

```bash
dpkg -s tree
```

Вивід:
```
Package: tree
Status: install ok installed
Priority: optional
Section: utils
Installed-Size: 113
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Architecture: amd64
Version: 2.0.2-1
Depends: libc6 (>= 2.34)
Description: displays an indented directory tree, in color
 Tree is a recursive directory listing command that produces a depth indented
 listing of files, which is colorized ala dircolors if the LS_COLORS environment
 variable is set and output is to tty.
Original-Maintainer: Florian Ernst <florian@debian.org>
Homepage: http://mama.indstate.edu/users/ice/tree/
```

(`Status: install ok installed` — пакет встановлено, `Version: 2.0.2-1` — встановлена версія.)

**Альтернативний спосіб — через сам бінарник:**

```bash
tree --version
```

Вивід:
```
tree v2.0.2 (c) 1996 - 2022 by Steve Baker, Thomas Moore, Francesc Rocher, Florian Sesser, Kyosuke Tokoro
```

**Видалити встановлений пакет:**

```bash
sudo apt remove -y tree
```

Вивід:
```
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  fonts-font-awesome htop libnl-3-200 libnl-genl-3-200 libntfs-3g89 libprotobuf23 libpython3.12-minimal
  libu2f-udev postgresql-16 postgresql-client-16 python3-asgiref python3-brotli python3-flask python3-h11
  python3-h2 python3-hpack python3-hyperframe python3-itsdangerous python3-kaitaistruct python3-ldap3
  python3-msgpack python3-passlib python3-protobuf python3-publicsuffix2 python3-pyinotify
  python3-pyperclip python3-ruamel.yaml python3-ruamel.yaml.clib python3-simplejson
  python3-sortedcontainers python3-tornado python3-urwid python3-werkzeug python3-wsproto
  python3.12-minimal
Use 'sudo apt autoremove' to remove them.
The following packages will be REMOVED:
  tree
0 upgraded, 0 newly installed, 1 to remove and 16 not upgraded.
After this operation, 116 kB disk space will be freed.
(Reading database ... 167336 files and directories currently installed.)
Removing tree (2.0.2-1) ...
Processing triggers for man-db (2.10.2-1) ...
```

**Перевірити, що пакета більше немає:**

```bash
dpkg -s tree
```

Вивід:
```
dpkg-query: package 'tree' is not installed and no information is available
Use dpkg --info (= dpkg-deb --info) to examine archive files.
```

---

## Завдання 2. Керування сервісами через `systemctl`

Працюватиму з сервісом `ssh`.

**Перевірити статус сервісу:**

```bash
sudo systemctl status ssh
```

Вивід:
```
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2026-05-07 13:52:51 CEST; 24h ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 524 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 537 (sshd)
      Tasks: 1 (limit: 28741)
     Memory: 12.7M
        CPU: 28min 1.191s
     CGroup: /system.slice/ssh.service
             └─537 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

May 08 14:10:34 vmi2706252.contaboserver.net sshd[192852]: Invalid user es from ***.***.***.*** port ***
May 08 14:10:34 vmi2706252.contaboserver.net sshd[192852]: Connection closed by invalid user es ***.***.***.*** port ***
```

**Зупинити сервіс і переконатися, що він не активний:**

```bash
sudo systemctl stop ssh
sudo systemctl status ssh
```

Вивід:
```
○ ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Fri 2026-05-08 10:27:05 EEST; 2s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 645 ExecStart=/usr/sbin/sshd -D $SSHD_OPTS (code=exited, status=0/SUCCESS)
   Main PID: 645 (code=exited, status=0/SUCCESS)
        CPU: 144ms

May 08 14:11:34 saniksin sshd[645]: Server listening on 0.0.0.0 port 22.
May 08 14:11:34 saniksin sshd[645]: Server listening on :: port 22.
```

**Запустити сервіс знову:**

```bash
sudo systemctl start ssh
sudo systemctl is-active ssh
```

Вивід:
```
active
```

**Додати сервіс в автозавантаження:**

```bash
sudo systemctl enable ssh
```

Вивід:
```
Synchronizing state of ssh.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable ssh
ssh.service is already enabled.
```

```bash
sudo systemctl is-enabled ssh
```

Вивід:
```
enabled
```

---

## Завдання 3. Робота з логами

**Перейти в `/var/log` та вивести останні 10 рядків `syslog`:**

```bash
cd /var/log
sudo tail -n 10 syslog
```

Вивід:
```
May  8 14:07:21 saniksin systemd-resolved[277]: message repeated 4 times: [ Clock change detected. Flushing caches.]
May  8 14:07:52 saniksin systemd-resolved[277]: Clock change detected. Flushing caches.
May  8 14:09:56 saniksin systemd-resolved[277]: message repeated 4 times: [ Clock change detected. Flushing caches.]
May  8 14:10:27 saniksin systemd-resolved[277]: Clock change detected. Flushing caches.
May  8 14:10:58 saniksin systemd-resolved[277]: Clock change detected. Flushing caches.
May  8 14:13:03 saniksin systemd-resolved[277]: message repeated 4 times: [ Clock change detected. Flushing caches.]
May  8 14:13:34 saniksin systemd-resolved[277]: Clock change detected. Flushing caches.
May  8 14:14:36 saniksin systemd-resolved[277]: message repeated 2 times: [ Clock change detected. Flushing caches.]
May  8 14:15:01 saniksin CRON[3806589]: (root) CMD (command -v debian-sa1 > /dev/null && debian-sa1 1 1)
May  8 14:15:07 saniksin systemd-resolved[277]: Clock change detected. Flushing caches.
```

**Переглянути тільки помилки в системі через `journalctl` (priority `err`):**

```bash
sudo journalctl -p err -b
```

Вивід:
```
-- Journal begins at Fri 2026-05-01 08:12:03 EEST, ends at Fri 2026-05-08 10:28:14 EEST. --
May 08 09:14:18 saniksin kernel: ACPI BIOS Error (bug): Could not resolve symbol [\_SB.PCI0.GFX0._DSM], AE_NOT_FOUND
May 08 09:14:19 saniksin systemd[1]: Failed to start Hyper-V File System Shares.
May 08 09:14:21 saniksin bluetoothd[523]: Failed to set mode: Blocked through rfkill (0x12)
```

(`-p err` обмежує видачу рівнями `emerg/alert/crit/err`; `-b` — тільки поточне завантаження.)

або
```bash
sudo journalctl -u ssh -p err -b
```

Вивід:
```
-- No entries --
```

**Знайти у журналах запис про запуск/зупинку сервісу `ssh`:**

```bash
sudo journalctl -u ssh --since "1 hour ago" | grep -E "Started|Stopped"
```

Вивід:
```
May 08 14:11:34 saniksin systemd[1]: Stopped OpenBSD Secure Shell server.
May 08 14:11:34 saniksin systemd[1]: Started OpenBSD Secure Shell server.
```

---

## Завдання 4. Створення власного сервісу

**Створити простий bash-скрипт у домашньому каталозі — щосекунди записує поточну дату у текстовий файл:**

```bash
nano ~/date_writer.sh
```

Вміст файлу `~/date_writer.sh`:
```bash
#!/usr/bin/env bash
# Кожну секунду дописує поточну дату у файл /home/saniksin/date_log.txt

OUTPUT="/home/saniksin/date_log.txt"

while true; do
    date '+%Y-%m-%d %H:%M:%S' >> "$OUTPUT"
    sleep 1
done
```

**Зробити скрипт виконуваним:**

```bash
chmod +x ~/date_writer.sh
ls -l ~/date_writer.sh
```

Вивід:
```
-rwxr-xr-x 1 saniksin saniksin 241 May  8 14:19 /home/saniksin/date_writer.sh
```

**Створити файл конфігурації сервісу `/etc/systemd/system/myscript.service`:**

```bash
sudo nano /etc/systemd/system/myscript.service
```

Вміст файлу `/etc/systemd/system/myscript.service`:
```ini
[Unit]
Description=Date writer service - writes current date every second
After=network.target

[Service]
Type=simple
ExecStart=/home/saniksin/date_writer.sh
Restart=on-failure
User=saniksin
Group=saniksin

[Install]
WantedBy=multi-user.target
```

**Перечитати конфігурацію `systemd` та запустити сервіс:**

```bash
sudo systemctl daemon-reload
sudo systemctl start myscript.service
```

**Перевірити статус сервісу:**

```bash
sudo systemctl status myscript.service
```

Вивід:
```
● myscript.service - Date writer service - writes current date every second
     Loaded: loaded (/etc/systemd/system/myscript.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2026-05-08 14:22:35 CEST; 14s ago
   Main PID: 3816796 (bash)
      Tasks: 2 (limit: 37069)
     Memory: 784.0K
     CGroup: /system.slice/myscript.service
             ├─3816796 bash /home/saniksin/date_writer.sh
             └─3817122 sleep 1

May 08 14:22:35 saniksin systemd[1]: Started Date writer service - writes current date every second.
```

**Переконатися, що дані записуються у файл:**

```bash
tail -n 5 ~/date_log.txt
```

Вивід:
```
2026-05-08 14:23:01
2026-05-08 14:23:02
2026-05-08 14:23:03
2026-05-08 14:23:04
2026-05-08 14:23:05
```

**Додати сервіс в автозавантаження:**

```bash
sudo systemctl enable myscript.service
```

Вивід:
```
Created symlink /etc/systemd/system/multi-user.target.wants/myscript.service → /etc/systemd/system/myscript.service.
```

**Зупинити сервіс після перевірки:**

```bash
sudo systemctl disable myscript.service
sudo systemctl stop myscript.service
sudo systemctl is-active myscript.service
```

Вивід:
```
inactive
```
