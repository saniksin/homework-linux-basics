# Домашнє завдання №6. Bash-скрипт бекапу логів

**Автор:** Македон

Обрано **Варіант A — скрипт бекапу логів**.

---

## Завдання. Скрипт `backup.sh`

### Код скрипта

```bash
#!/usr/bin/env bash
#
# Назва скрипта: backup.sh
# Опис:          Резервне копіювання (бекап) усіх файлів з каталогу логів
#                у стиснений tar.gz-архів з міткою дати й часу в імені.
# Використання:  ./backup.sh <log_dir> <backup_dir>
# Автор:         Олександр Македон
# Дата:          2026-06-02
#

set -u                          # звернення до невизначеної змінної — помилка

LOCKFILE="/tmp/backup.lock"     # lock-файл для захисту від паралельного запуску

# --- 1. Перевірка аргументів -------------------------------------------------
# Має бути рівно 2 аргументи, і обидва — існуючі каталоги.
if [[ $# -ne 2 || ! -d "$1" || ! -d "$2" ]]; then
    echo "Usage: ./backup.sh <log_dir> <backup_dir>"
    exit 1
fi

LOG_DIR="$1"
BACKUP_DIR="$2"

# --- 2. Захист від паралельного запуску --------------------------------------
# Якщо lock-файл уже існує — інший бекап у процесі, виходимо.
if [[ -e "$LOCKFILE" ]]; then
    echo "Backup already running"
    exit 1
fi

# Створюємо lock і гарантуємо його видалення при будь-якому виході зі скрипта.
touch "$LOCKFILE"
trap 'rm -f "$LOCKFILE"' EXIT

# --- 3. Створення архіву логів -----------------------------------------------
# Ім'я архіву містить дату й час: logs_backup_YYYY-MM-DD_HH-MM.tar.gz
TIMESTAMP="$(date '+%Y-%m-%d_%H-%M')"
ARCHIVE="${BACKUP_DIR}/logs_backup_${TIMESTAMP}.tar.gz"

# -C переходимо в каталог логів, '.' — усі файли всередині (без шляху-префікса).
tar -czf "$ARCHIVE" -C "$LOG_DIR" . 2>/dev/null
rc=$?                           # зберігаємо код повернення tar одразу

# --- 4. Перевірка результату -------------------------------------------------
if [[ $rc -ne 0 ]]; then
    echo "Backup failed"
    exit 2
fi

echo "Backup created: $(realpath "$ARCHIVE")"
```

### Короткий опис, що відбувається

- **Заголовок-docstring** на початку файлу (Назва / Опис / Використання / Автор / Дата) — стандарт документування скриптів, щоб призначення було зрозумілим без читання коду.
- **`set -u`** — захист від помилок через незаданi змінні (best practice).
- **Блок 1** перевіряє одночасно три умови: кількість аргументів (`$# -ne 2`) та існування обох каталогів (`! -d`). За будь-якого порушення друкує підказку й завершується з кодом `1`.
- **Блок 2** реалізує lock через `/tmp/backup.lock`. Якщо файл є — другий екземпляр не запускається. `trap '... EXIT'` прибирає lock автоматично при нормальному завершенні, помилці чи перериванні — навіть якщо архівація впаде.
- **Блок 3** формує ім'я з міткою часу через `date` і пакує вміст каталогу логів у `tar.gz`. Прапорець `-C` гарантує, що в архіві лежать самi файли, а не повний шлях.
- **Блок 4** аналізує код повернення `tar`: ненульовий → `Backup failed` + `exit 2`; успіх → друк абсолютного шляху через `realpath`.

---

### Перевірка роботи з різними аргументами

**Підготовка: робимо скрипт виконуваним і створюємо тестові дані:**

```bash
chmod +x backup.sh
ls -l backup.sh
```

Вивід:
```
-rwxr-xr-x 1 saniksin saniksin 1357 Jun  2 18:40 backup.sh
```

```bash
mkdir -p ~/demo/logs ~/demo/backups
echo "app started"   > ~/demo/logs/app.log
echo "warning: low"  > ~/demo/logs/error.log
echo "GET / 200"     > ~/demo/logs/access.log
ls -l ~/demo/logs
```

Вивід:
```
total 12
-rw-r--r-- 1 saniksin saniksin 10 Jun  2 18:40 access.log
-rw-r--r-- 1 saniksin saniksin 12 Jun  2 18:40 app.log
-rw-r--r-- 1 saniksin saniksin 11 Jun  2 18:40 error.log
```

**Тест 1 — без аргументів (очікуємо Usage та код 1):**

```bash
./backup.sh
echo "Exit code: $?"
```

Вивід:
```
Usage: ./backup.sh <log_dir> <backup_dir>
Exit code: 1
```

**Тест 2 — лише один аргумент:**

```bash
./backup.sh ~/demo/logs
echo "Exit code: $?"
```

Вивід:
```
Usage: ./backup.sh <log_dir> <backup_dir>
Exit code: 1
```

**Тест 3 — другий каталог не існує:**

```bash
./backup.sh ~/demo/logs ~/demo/nonexistent
echo "Exit code: $?"
```

Вивід:
```
Usage: ./backup.sh <log_dir> <backup_dir>
Exit code: 1
```

**Тест 4 — коректний запуск (успішний бекап):**

```bash
./backup.sh ~/demo/logs ~/demo/backups
echo "Exit code: $?"
```

Вивід:
```
Backup created: /home/saniksin/demo/backups/logs_backup_2026-06-02_18-42.tar.gz
Exit code: 0
```

Перевіряємо, що архів справді створено і містить усі логи:

```bash
ls -lh ~/demo/backups
tar -tzf ~/demo/backups/logs_backup_2026-06-02_18-42.tar.gz
```

Вивід:
```
total 4.0K
-rw-r--r-- 1 saniksin saniksin 178 Jun  2 18:42 logs_backup_2026-06-02_18-42.tar.gz
./
./access.log
./app.log
./error.log
```

Архів містить усі три файли логів. Lock-файл після завершення прибрано автоматично (`trap`):

```bash
ls -l /tmp/backup.lock
```

Вивід:
```
ls: cannot access '/tmp/backup.lock': No such file or directory
```

**Тест 5 — захист від паралельного запуску:**

Імітуємо, що інший екземпляр уже працює (lock-файл існує):

```bash
touch /tmp/backup.lock
./backup.sh ~/demo/logs ~/demo/backups
echo "Exit code: $?"
```

Вивід:
```
Backup already running
Exit code: 1
```

Прибираємо штучний lock після перевірки:

```bash
rm -f /tmp/backup.lock
```

(Вивід відсутній — файл видалено.)

**Тест 6 — невдала архівація (очікуємо Backup failed та код 2):**

Робимо каталог бекапів недоступним для запису, щоб `tar` не зміг створити архів:

```bash
chmod 555 ~/demo/backups
./backup.sh ~/demo/logs ~/demo/backups
echo "Exit code: $?"
```

Вивід:
```
Backup failed
Exit code: 2
```

Повертаємо права назад:

```bash
chmod 755 ~/demo/backups
```

(Вивід відсутній — права відновлено.)

---

### Підсумок завдання

- Скрипт `backup.sh` коректно обробляє **усі чотири вимоги** варіанта A:
  - **аргументи** — рівно 2 існуючих каталоги, інакше `Usage:` + код `1` (тести 1–3);
  - **lock** `/tmp/backup.lock` — захист від паралельного запуску, `Backup already running` (тест 5);
  - **архів** `logs_backup_YYYY-MM-DD_HH-MM.tar.gz` з усіма логами (тест 4);
  - **перевірка результату** — `Backup created: <повний шлях>` при успіху (код `0`) та `Backup failed` + код `2` при помилці (тест 6).
- Best practice: `set -u`, `trap` для гарантованого зняття lock, збереження `rc=$?` одразу після `tar`, коментарі до кожного блоку.
