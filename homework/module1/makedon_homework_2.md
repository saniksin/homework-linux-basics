# Домашнє завдання №2. Файлова система і права доступу

## Завдання 1. Ієрархія каталогів Linux

**Перейти в кореневий каталог `/` і показати вміст:**

```bash
cd /
ls
```

Вивід:
```
bin   dev  home  lib32  libx32      media  opt   root  sbin  srv  tmp  var
boot  etc  lib   lib64  lost+found  mnt    proc  run   snap  sys  usr
```

**Перейти в `/etc` і показати вміст:**

```bash
cd /etc
ls
```

Вивід:
```
adduser.conf     dpkg             hostname     ldap            passwd          shadow-
alternatives     environment      hosts        locale.conf     passwd-         shells
apt              fonts            init.d       localtime       profile         skel
bash.bashrc      fstab            inputrc      login.defs      profile.d       ssh
bash_completion  group            issue        logrotate.conf  rc0.d           ssl
ca-certificates  group-           issue.net    lsb-release     resolv.conf     sudoers
crontab          gshadow          kernel       machine-id      security        sudoers.d
cron.d           gshadow-         ld.so.cache  mime.types      services        sysctl.conf
cron.daily       hostid           ld.so.conf   netplan         shadow          systemd
```

**Перейти у каталог `/home` і показати список користувачів:**

```bash
cd /home
ls
```

Вивід:
```
saniksin
```

---

## Завдання 2. Файли, каталоги та посилання

**Створити новий каталог у домашньому каталозі:**

```bash
mkdir ~/lab2
```

**Створити всередині файл:**

```bash
touch ~/lab2/file.txt
```

**Скопіювати файл у нове ім'я:**

```bash
cp ~/lab2/file.txt ~/lab2/file_copy.txt
```

**Перейменувати копію:**

```bash
mv ~/lab2/file_copy.txt ~/lab2/file_renamed.txt
```

**Створити жорстке посилання:**

```bash
ln ~/lab2/file.txt ~/lab2/file_hardlink.txt
```

**Створити символічне посилання:**

```bash
ln -s ~/lab2/file.txt ~/lab2/file_symlink.txt
```

**Знайти файл по імені:**

```bash
find / -name "file.txt" 2>/dev/null
```

Вивід:
```
/home/saniksin/lab2/file.txt
```

---

## Завдання 3. Права доступу

**Переглянути права файлу, який ти створив:**

```bash
ls -l ~/lab2/file.txt
```

Вивід:
```
-rw-r--r-- 2 saniksin saniksin 0 Apr 21 12:00 /home/saniksin/lab2/file.txt
```

**Надати файлу права тільки на читання:**

```bash
chmod 444 ~/lab2/file.txt
ls -l ~/lab2/file.txt
```

Вивід:
```
-r--r--r-- 2 saniksin saniksin 0 Apr 21 12:00 /home/saniksin/lab2/file.txt
```

**Надати власнику право на запис:**

```bash
chmod u+w ~/lab2/file.txt
ls -l ~/lab2/file.txt
```

Вивід:
```
-rw-r--r-- 2 saniksin saniksin 0 Apr 21 12:00 /home/saniksin/lab2/file.txt
```

**Переглянути значення `umask`:**

```bash
umask
```

Вивід:
```
0002
```

**Встановити нове значення, наприклад `022`:**

```bash
umask 022
umask
```

Вивід:
```
0022
```

---

## Завдання 4. Користувачі

**Створити нового користувача:**

```bash
sudo adduser trainee
```

Вивід:
```
Adding user `trainee' ...
Adding new group `trainee' (1001) ...
Adding new user `trainee' (1001) with group `trainee' ...
Creating home directory `/home/trainee' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for trainee
Enter the new value, or press ENTER for the default
        Full Name []:
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
Adding new user `trainee' to supplemental / extra groups `users' ...
Adding user `trainee' to group `users' ...
```

**Додати його до sudo-групи:**

```bash
sudo usermod -aG sudo trainee
```

**Перевірити, що користувач існує:**

```bash
cat /etc/passwd | grep trainee
```

Вивід:
```
trainee:x:1001:1001::/home/trainee:/bin/bash
```

```bash
id trainee
```

Вивід:
```
uid=1001(trainee) gid=1001(trainee) groups=1001(trainee),27(sudo),100(users)
```
