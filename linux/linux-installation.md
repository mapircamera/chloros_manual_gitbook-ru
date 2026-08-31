# Установка Linux

Chloros распространяется для Linux в виде пакетов `.deb`, которые устанавливают CLI и бэкэнд-сервер. Python SDK представляет собой отдельный пакет pip (также входящий в состав `.deb` в виде файла wheel, соответствующего версии).

Имена файлов пакетов содержат версию и архитектуру: `chloros_1.2.0_amd64.deb` для x86_64 и `chloros_1.2.0_arm64_jp6.deb` для сборок JetPack 6 Jetson. В приведенных ниже командах замените файл на тот, который вы фактически скачали.

***

## Linux amd64 (x86_64)

### Системные требования

| Требование | Минимальные | Рекомендуемые |
| --- | --- | --- |
| **Дистрибутив** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Процессор** | x86_64 (Intel/AMD) | Intel Core i7 или лучше |
| **Оперативная память (RAM)** | 8 ГБ | 16 ГБ или больше |
| **Видеокарта** | Не требуется (обработка на процессоре) | Видеокарта NVIDIA с 4 ГБ и более видеопамяти (12 ГБ и более разблокируют `GPU_PARALLEL`, 7 ГБ и более позволяют отключить Texture Aware в режиме обработки одного изображения) |
| **Место на диске** | 2 ГБ свободного места | SSD с 10 ГБ и более свободного места |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

> **Ubuntu 20.04 и Debian 11 не поддерживаются.** Список зависимостей `.deb`
> основан на том, с чем фактически связывается бэкенд Chloros, и в него входит
> `libc6 (>= 2.34)`. В дистрибутивах Focal и bullseye используется glibc 2.31, поэтому `apt` сразу отклоняет
> установку, чтобы избежать сбоев позже во время выполнения.

### Установка

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
`dpkg -i` не разрешает зависимости. Если он сообщает об отсутствующих пакетах, `sudo apt-get install -f` (или `sudo apt --fix-broken install`) завершает установку — это нормальный процесс, а не ошибка.
{% endhint %}

Проверьте установку:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Системные требования

| Требование | Минимальное | Рекомендуемое |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson с JetPack 6 | Jetson Orin NX 16 ГБ или AGX Orin |
| **JetPack** | JetPack 6.x | Последняя версия JetPack 6 |
| **Память (RAM)** | 8 ГБ (совместное использование GPU/CPU) | 16 ГБ и более (совместное использование) (12 ГБ и более — пороговое значение для параллельных рабочих процессов GPU) |
| **Хранилище** | 2 ГБ свободного места | SSD NVMe с 10 ГБ+ свободного места |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

### Установка

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Такая же конфигурация, как у amd64 `.deb`, с сборкой CUDA, оптимизированной для Jetson Orin / Orin NX / Orin Nano. Информацию о памяти, тепловых характеристиках и поведении при развертывании в полевых условиях Jetson см. в [Руководстве по NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Установка Python и SDK (все версии Linux)

SDK — это чистый клиент Python HTTP для бэкенда, поэтому один и тот же пакет работает как на amd64, так и на arm64. Два источника:**С PyPI** — опубликованный стабильный выпуск:

```bash
pip install chloros-sdk
```

**Из входящего в комплект файла wheel** — гарантированно совместим с CLI/backend, который вы только что установили (используйте его, если ваша версия `.deb` новее, чем на PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Дистрибутивы, соответствующие PEP 668** (Ubuntu 23.10+, Debian 12+) не допускают установку pip в системе в целом. Используйте `pip install --user …`, виртуальную среду или `sudo pip install --break-system-packages …`. Установщик пакетов никогда не устанавливает автоматически SDK в вашу систему Python — этот выбор остается за вами.
{% endhint %}

Дополнительные компоненты:

| Дополнение | Команда | Добавляет |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` для потоковой передачи данных о ходе выполнения в реальном времени |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` для передачи данных по BLE (DAQ-M) |

Проверьте SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
`.deb` устанавливает Chloros, CLI и бэкэнд. Python и SDK взаимодействуют с этим бэкендом через локальный HTTP и API (`http://127.0.0.1:5000`) и автоматически запускает его при необходимости. Всегда используйте буквальный IPv4-адрес, а не `localhost` — `localhost` может разрешаться в `::1` и занимать примерно две секунды на каждый запрос.
{% endhint %}

***

## Первоначальная настройка

### 1. Вход в систему

Для доступа к CLI и SDK требуется платный тарифный план Chloros+ (**Copper** или выше), что проверяется на стороне сервера: вызов без авторизации получает код `401 AUTH_REQUIRED`, а вызов с бесплатным тарифом (Iron) — `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Учетные данные кэшируются в `~/.chloros/user_session.json`.

{% hint style="warning" %}
**После каждой установки или обновления необходимо заново войти в систему.** Скрипт `prerm`, входящий в состав пакета, намеренно очищает `~/.chloros/user_session.json` и кэшированную лицензию для каждого пользователя на компьютере, поэтому новая сборка всегда заново проверяет лицензию, а не полагается на устаревший кэш.
{% endhint %}

### 2. Проверьте статус лицензии

```bash
chloros-cli status
```

`chloros-cli status` работает на любом уровне (включая бесплатный), поэтому вы всегда можете понять, почему доступ доступен или недоступен.

### 3. Запустите диагностику системы

```bash
chloros-cli selftest
```

Семь проверок выполняются по порядку, и команда завершается с ненулевым кодом, если какая-либо из них завершилась с ошибкой:

| # | Проверка | Что она подтверждает |
| --- | --- | --- |
| 1 | **Версия** | CLI сообщает свою версию (`v1.2.0`). |
| 2 | **Порт доступен** | Порт 5000 свободен *или* на него уже ответил исправный бэкенд Chloros (что считается успешным прохождением). |
| 3 | **Запуск бэкенда** | Запускается бинарный файл бэкенда. |
| 4 | **Тест API (`/api/test`)** | Бэкенд отвечает `status: ok`. |
| 5 | **Информация о системе** | Выводит `GPU: <name>, CUDA: <bool>, PyTorch: <version>` из `/api/system-info`. |
| 6 | **Модели шумоподавителя** | Находит модели `*.pth.enc` (в Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + фильтр шума**| Функция Texture Aware действительно работает — требует CUDA**и** как минимум одного файла модели. |

Выполнение заканчивается на `N/7 checks passed` с перечнем всех сбоев по именам.

### 4. Обработка вашего первого набора данных

```bash
chloros-cli process ~/datasets/flight001
```

***

## Файлы и каталоги

### Пользовательские

Chloros хранит свои учетные данные и конфигурацию CLI в одном кроссплатформенном каталоге **`~/.chloros/`** (в Windows, `%USERPROFILE%\.chloros\`). Два кэша, специфичных для Linux, вместо этого следуют конвенциям XDG — они учитывают настройки `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`, если они заданы.

| Путь | Назначение |
| --- | --- |
| `~/.chloros/user_session.json` | Кэш сеанса входа, записываемый `chloros-cli login` (очищается при каждой установке/обновлении пакета) |
| `~/.chloros/working_directory.txt` | Переопределение папки проекта по умолчанию (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Настройка языка CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Настройка языка, общая с графическим интерфейсом Windows — значение `language` имеет приоритет над `cli_language.json` |
| `~/.chloros/update_cache.json` | Одночасовой кэш для проверки обновлений при запуске Linux/Jetson |
| `~/.chloros/backend.log` | Журнал бэкэнда при его запуске CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Кэшированные пакеты калибровки LATTICE для каждой камеры, с ключом в виде серийного номера и хеша пакета |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Дополнительные пользовательские настройки для профилей коррекции капа DAQ |
| `~/.config/chloros/system_config.json` | Кэшированный профиль оборудования из Dynamic Compute Adaptation — удалите его, чтобы принудительно запустить новое обнаружение оборудования |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Журналы бэкенд-сервера, по одному файлу на каждый запуск |
| `~/Chloros Projects/` | Папка проекта по умолчанию, если переопределение не задано |

### Системные

| Путь | Назначение |
| --- | --- |
| `/usr/bin/chloros-cli` | Скрипт-обёртка — устанавливает `LD_LIBRARY_PATH` для входящих в комплект нативных библиотек, а затем запускает собственно исполняемый файл |
| `/usr/bin/chloros-backend` | Скрипт-обёртка — то же самое, плюс `CHLOROS_PRODUCTION=1`, чтобы механизм авторизации бэкэнда никогда не мог незаметно отключиться |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Скомпилированные бинарные файлы |
| `/usr/lib/chloros/arena_runtime/` | Среда выполнения Arena SDK, необходимая для камер LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Зашифрованные модели шумоподавителя, используемые дебайером Texture Aware |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python SDK — набор, соответствующий именно этой сборке |
| `/usr/lib/chloros/exiftool` | Включённый в комплект exiftool (символьная ссылка на `/usr/local/bin/exiftool` создаётся только в случае отсутствия системного exiftool) |
| `/etc/chloros/update.conf` | Конфигурация канала обновлений, считываемая `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Настраивает `net.ipv4.ip_unprivileged_port_start = 319`, чтобы бэкенд мог привязывать порты PTP без прав root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Направляет динамический загрузчик на `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Предоставляет вошедшему в систему пользователю доступ к USB-мосту DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Включает постоянно работающую службу бэкэнда (установлена, **не включена**) |
| `/usr/share/applications/chloros-cli.desktop` | Пункт меню приложения «Chloros CLI», открывающий терминал |

## Расположение исполняемого файла бэкэнда

CLI и SDK автоматически определяют бэкэнд:

| Компонент | Путь |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Бэкенд | `/usr/lib/chloros/chloros-backend` |

Переопределите путь к бэкенду с помощью флага `--backend-exe` CLI или параметра конструктора `backend_exe` SDK, а порт — с помощью `--port` (по умолчанию — `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` указывает на **`lattice`**,**`project`**и**`daq pool-*`** на удалённый бэкенд. Основные команды (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) намеренно игнорируют его и всегда нацеливаются на `http://127.0.0.1:<port>`.
{% endhint %}

***

## Камеры LATTICE и световые датчики DAQ на Linux

Все семейства команд live-hardware работают на Linux (amd64 и Jetson):

* **`chloros-cli lattice`** — обнаружение, подключение, настройка, и захват данных с камер LATTICE и синхронизированных массивов. `.deb` включает в себя необходимую среду выполнения Arena SDK и регистрирует её в динамическом загрузчике.
* **`chloros-cli daq pool-*`** — подключение датчиков освещенности DAQ-U/M/E через пул бэкэндов, потоковая передача откалиброванных спектров и запись файлов `.daq`. Скомпилированный CLI поставляется только с семейством `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — запускает сохраненный проект (его камеры, датчики и настройки обработки) в режиме без интерфейса.
* **`chloros-cli time-sync`** — проверка главного сервера PTP, на котором работает бэкенд Chloros для камер LATTICE и датчиков DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` требуется для работы `pool-latest`, `pool-stream`, `pool-record` и `pool-set-cap`; `pool-list` показывает идентификаторы, которые в данный момент находятся в пуле.

{% hint style="info" %}
**Для первого подключения DAQ-E на машине с несколькими сетевыми интерфейсами предпочтительно использовать `--eth-host`.** Функция автоматического обнаружения просматривает mDNS и может пропустить интерфейс датчика из-за пустого кэша ARP, поэтому первое подключение `pool-connect --eth` после загрузки может завершиться неудачей, даже если датчик находится в идеальном рабочем состоянии. Передача IP-адреса или имени хоста датчика позволяет полностью обойти процесс обнаружения.
{% endhint %}

**Права доступа к последовательному интерфейсу DAQ-U** регулируются установленным правилом udev (`uaccess` + группа `dialout`). Если датчик, который уже был подключен, остаётся недоступным, перезагрузите правила или подключите его заново:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Полный набор команд см. в [справочнике по CLI](../CLI.md).

### Постоянно включенный PTP для хостов без монитора

При первоначальной установке модуль systemd `chloros-backend.service` создаётся, но **не включается**. На безмониторном устройстве Jetson или сервере, где должна постоянно работать синхронизация времени PTP для датчиков DAQ-E и камер LATTICE, включите его:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Без него PTP работает только при запущенном бэкенде Chloros — то есть во время активной сессии CLI/SDK.

Устройство привязывает бэкенд к `127.0.0.1:5000` (настройки среды `CHLOROS_HOST` / `CHLOROS_PORT` внутри устройства; переопределяются с помощью `sudo systemctl edit chloros-backend.service`) и перезапускает его в случае сбоя через 5 секунд.

**Как PTP получает свои порты.** PTP использует UDP 319/320, оба из которых находятся ниже обычного нижнего предела привилегированных портов (1024). Файл `postinst` из пакета записывает `/etc/sysctl.d/60-chloros-ptp.conf` с помощью `net.ipv4.ip_unprivileged_port_start = 319`, что позволяет бэкенду привязаться к ним при работе от имени вашего пользователя. Кроме того, в качестве дополнительной меры предосторожности к бинарному файлу бэкэнда применяется `setcap cap_net_bind_service,cap_net_raw=+ep` — именно поэтому `libcap2-bin` является объявленной зависимостью пакета.***

## Примеры скриптов Bash

{% hint style="info" %}
**Коды завершения, удобные для скриптов.**`chloros-cli process` возвращает код `0` при успешном завершении и**код, отличный от нуля, при сбое — включая запуск, при котором запрашивались продукты изображений, но ни один из них не был записан** (он выводит код `Processing finished but wrote no image products.`, указывает имя папки проекта и типичные причины). При успешном выполнении сообщается, сколько изображений было записано (`Image products written: N`). Коды завершения: `0` — успех, `1` — сбой, `2` — ошибка аргумента, `130` — прервано.
{% endhint %}

### Обработка нескольких наборов данных

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Обработка с пользовательскими настройками

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Допустимых значений `--format` ровно четыре, и они содержат пробелы — всегда заключайте их в кавычки:

| Значение `--format` | Папка вывода |
| --- | --- |
| `TIFF (16-bit)` *(по умолчанию)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` принимает `standard` (по умолчанию) или `texture-aware` (Chloros+).

### Автоматическая обработка с помощью Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Пример с Python и SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Устранение неполадок

### CLI не найден после установки

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Отказ в доступе

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### «setcap failed» во время установки

`.deb` применяет `cap_net_bind_service` к `/usr/lib/chloros/chloros-backend`, чтобы тот мог привязывать PTP-порты 319/320 без прав root. Если во время установки отсутствовал файл `libcap2-bin`, вызов пропускается. Установите его и переустановите пакет:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### PTP не запускается / не может привязаться к порту 319

Убедитесь, что нижний порог для портов без привилегий был снижен, и, если это не было сделано, примените его заново для текущей загрузки:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Затем проверьте главный сервер:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### «Не найдены драйверы камер LATTICE»

Не удаётся разрешить среду выполнения Arena SDK. Убедитесь, что конфигурация загрузчика, записываемая пакетом, присутствует и обновлена:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
```

### Не удалось запустить бэкенд

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Журналы бэкенда, относящиеся к неудачному запуску, находятся в файле `~/.cache/chloros/logs/`.

### CUDA не обнаружена

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

`chloros-cli selftest` сообщает об этом же в одной строке: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Отсутствуют общие библиотеки

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Медленный запуск на системах с SD-картами

Скомпилированные бинарные файлы при каждом запуске извлекают себя во временный каталог. Если файл `/mnt/ssd/tmp` существует, Chloros использует его автоматически; в противном случае укажите `TMPDIR` как быструю файловую систему:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Обновление Chloros на Linux

Команда `update` доступна только на Linux/Jetson. Она проверяет версию, опубликованную в канале обновлений, настроенном в `/etc/chloros/update.conf`, и предлагает загрузить и установить соответствующий файл `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

На Linux/Jetson команда CLI также выполняет неблокирующую проверку обновлений при каждом запуске (результат кэшируется в течение одного часа в `~/.chloros/update_cache.json`) и выводит сообщение `Update available: vX.Y.Z` при наличии более новой версии. Ваши настройки и проекты сохраняются после обновления; после этого вам потребуется повторно войти в систему.

## Удаление

```bash
sudo apt remove chloros
```

Удаление останавливает работу `chloros-backend.service`, восстанавливает минимальное значение по умолчанию для непривилегированного порта (1024), удаляет символьную ссылку на встроенный exiftool и конфигурацию загрузчика Arena, а также очищает кэшированные учетные данные. Ваши проекты и файлы данных `~/.chloros/` остаются неизменными.

***

## Следующие шаги

* [Руководство по NVIDIA Jetson](nvidia-jetson-guide.md) — оптимизация и развёртывание специально для Jetson
* [CLI : Командная строка](../CLI.md) — руководство по CLI
* [API : Python SDK](../api-python-sdk.md) — руководство по SDK
* [Справочник по CLI](../reference/cli-reference.md) и [Справочник по SDK](../reference/sdk-reference.md) — исчерпывающий список команд/API для версии 1.2.0
* [Динамическая адаптация вычислений](../processing-architecture/dynamic-compute-adaptation.md) — как Chloros адаптируется к вашему оборудованию
