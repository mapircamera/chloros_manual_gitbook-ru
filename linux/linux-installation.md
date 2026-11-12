# Установка в Linux

Chloros распространяется для Linux в виде пакетов `.deb`, которые устанавливают интерфейс CLI, бэкэнд-сервер и SDK для Python (для системы `python3`, а также соответствующий по версии файл wheel для virtualenvs).

Имена файлов пакетов содержат информацию о версии и архитектуре: `chloros_1.3.0_amd64.deb` для x86_64, и `chloros_1.3.0_arm64_jp6.deb` для arm64 (NVIDIA Jetson, JetPack 6). В приведённых ниже командах замените имя файла на то, которое вы фактически скачали.

{% hint style="info" %}
**Новое в версии 1.3.0: программы поставляются в распакованном виде.** Бэкенд и интерфейс командной строки (CLI) устанавливаются в виде папок — `/usr/lib/chloros/backend/` и `/usr/lib/chloros/cli/`, а не в виде отдельных самораспаковывающихся файлов. Они больше не распаковываются во временную папку при каждом запуске, поэтому запускаются гораздо быстрее (на x86_64 бэкенд раньше тратил около 100 секунд на распаковку при каждом запуске) и больше не требуют нескольких ГБ свободного места в папке `/tmp`. Размер папки amd64 `.deb` также сократился примерно вдвое по сравнению с прежним.
{% endhint %}

***

## Linux, amd64 (x86_64)

### Системные требования

| Требование | Минимальное | Рекомендуемое |
| --- | --- | --- |
| **Дистрибутив** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Процессор** | x86_64 (Intel/AMD) | Intel Core i7 или лучше |
| **Оперативная память (RAM)** | 8 ГБ | 16 ГБ или больше |
| **Видеокарта** | Не требуется (обработка на процессоре) | Видеокарта NVIDIA с 4 ГБ и более видеопамяти (12 ГБ и более разблокируют `GPU_PARALLEL`, 7 ГБ и более позволяют отключить Texture Aware в режиме обработки одного изображения) |
| **Хранение** | 2 ГБ свободного места | SSD с 10 ГБ и более свободного места |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

> **Ubuntu 20.04 и Debian 11 не поддерживаются.** Список зависимостей `.deb`
> составлен на основе того, с чем фактически связывается бэкенд Chloros, и включает
> версию `libc6`, более новую, чем glibc 2.31, поставляемая в дистрибутивах focal и bullseye, поэтому `apt` сразу же отказывает в
> установке, вместо того чтобы допустить сбой позже во время выполнения.

### Установка

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Используйте `apt install ./…`, а не `dpkg -i`.** `apt` разрешает зависимости пакета и также устанавливает то, что он рекомендует: требования Python SDK, `python3-numpy` и `python3-requests`. `dpkg -i` не устанавливает ни того, ни другого. CLI-интерфейс и бэкенд являются автономными и работают без них, но `import chloros_sdk` нуждается в них; в выводе установки указаны все отсутствующие компоненты. Сохраните `./`, чтобы `apt` установил файл, а не искал пакет с таким именем. Пакет также объявляет `libgl1`, который загружает входящий в комплект OpenCV и которого может не хватать в минимальном образе сервера.
{% endhint %}

Проверьте установку:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Системные требования

| Требование | Минимальное | Рекомендуемое |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson с JetPack 6 | Jetson Orin NX 16 ГБ или AGX Orin |
| **JetPack** | JetPack 6.x | Последняя версия JetPack 6 |
| **Память (RAM)** | 8 ГБ (общая для GPU/CPU) | 16 ГБ+ общей памяти (12 ГБ+ — пороговое значение для параллельной обработки) |
| **Хранение** | 2 ГБ свободного места | SSD NVMe с 10 ГБ+ свободного места |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

### Установка

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Такая же конфигурация, как и у amd64 `.deb` (включает Python SDK, буферы приёма и бэкенд-сервисы), для Jetson Orin / Orin NX / Orin Nano на **JetPack 6** (L4T R36). Используется версия PyTorch, скомпилированная с CUDA для Orin в рамках JetPack 6, поэтому обработка на GPU, включая дебайеризатор с поддержкой текстур, выполняется на графическом процессоре Jetson. Сам пакет не содержит CUDA: он использует библиотеки CUDA и cuDNN, устанавливаемые JetPack 6 (`sudo apt install nvidia-jetpack` на образе, в котором их нет). Если графический процессор с поддержкой CUDA недоступен, обработка выполняется на ЦП, и `--debayer texture-aware` использует стандартный дебайер (бэкенд записывает в журнал одну строку `[DEBAYER] … needs a CUDA GPU …` с объяснением причины). Информацию о поведении Jetson в отношении памяти, тепловыделения и развертывания в полевых условиях см. в [Руководстве по NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Установка Python SDK (для всех версий Linux)**`.deb` устанавливает SDK автоматически.** Оба пакета (amd64 и Jetson) устанавливают его для системной среды `python3`, поэтому `import chloros_sdk` работает сразу после установки, без pip и без подключения к сети. Его пакеты (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` и загрузчики проектов) помещаются в `/usr/lib/python3/dist-packages`, принадлежащий данному пакету: при обновлении они заменяются, а при удалении пакета — удаляются. Версия всегда соответствует установленным вами интерфейсу CLI и бэкенду.

Собственные зависимости SDK, `python3-numpy` и `python3-requests`, **рекомендуются, но не являются обязательными** для данного пакета: `sudo apt install ./chloros_<version>_<arch>.deb` устанавливает их, а `dpkg -i` — нет. В выводе установки указывается версия SDK, которую импортирует `python3`, любые отсутствующие компоненты, а также о любом файле `chloros_sdk`, находящемся в другом месте в пути `python3` и перекрывающем копию из пакета (например, установленном ранее с помощью `pip install --user`).**В virtualenv**, который не видит системные пакеты, установите в него входящий в комплект файл wheel (полностью в автономном режиме):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**С PyPI** — только для компьютера, на котором не установлен пакет Chloros, чьё SDK управляет бэкендом на другом компьютере (см. [Режим удалённого бэкенда](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Дополнительные компоненты для установки с помощью pip (в virtualenv или из PyPI):

| Дополнение | Команда | Добавляет |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` для потоковой передачи прогресса в реальном времени |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` для транспорта BLE (DAQ-M) |

Проверьте SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Python-SDK взаимодействует с бэкендом, который `.deb` устанавливает через локальный HTTP-API (`http://127.0.0.1:5000`). `ChlorosLocal` и вспомогательные программы подключения запускают его автоматически при необходимости; `connect_all()` сохраненного проекта никогда этого не делает, поэтому на машине без монитора запускайте бэкенд в качестве [службы](#always-on-ptp-for-headless-hosts). Всегда используйте буквальный IPv4-адрес, а не `localhost` — `localhost` может разрешаться в `::1` и занимать примерно две секунды на каждый запрос.
{% endhint %}

***

## Первоначальная настройка

### 1. Вход в систему

Для доступа к CLI и SDK требуется платный тариф Chloros+ (**Copper** или выше), что обеспечивается на стороне сервера: вызов без авторизации получает адрес `401 AUTH_REQUIRED`, а вызов с бесплатным тарифом (Iron) — `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

Учетные данные кэшируются в `~/.chloros/user_session.json`.

{% hint style="info" %}
**При обновлении учетные данные сохраняются; при удалении они очищаются.** Установка более новой версии `.deb` поверх существующей установки (в том числе через `chloros-cli update`) сохраняет `~/.chloros/user_session.json` и кэшированную лицензию, поэтому задания CLI/SDK, выполняемые в режиме без графического интерфейса, продолжают работать. Удаление пакета (`sudo apt remove chloros`) очищает сохраненные данные входа в систему и кэш лицензии для всех пользователей на компьютере, поэтому для новой установки потребуется снова установить `chloros-cli login`.
{% endhint %}

### 2. Проверьте статус лицензии

```bash
chloros-cli status
```

`chloros-cli status` работает на любом тарифном плане (включая бесплатный), поэтому вы всегда можете увидеть, почему доступ доступен или недоступен.

### 3. Запустите диагностику системы

```bash
chloros-cli selftest
```

Семь проверок выполняются по порядку, и команда завершается с ненулевым кодом, если какая-либо из них завершилась с ошибкой:

| # | Проверка | Что она подтверждает |
| --- | --- | --- |
| 1 | **Версия** | Интерфейс CLI сообщает свою версию (`v1.3.0`). |
| 2 | **Порт доступен** | Порт 5000 свободен *или* на него уже ответил исправный бэкенд Chloros (что считается успешным прохождением). |
| 3 | **Запуск бэкенда** | Запускается бинарный файл бэкенда. |
| 4 | **Тест API (`/api/test`)** | Бэкенд отвечает `status: ok`. |
| 5 | **Информация о системе** | Выводит `GPU: <name>, CUDA: <bool>, PyTorch: <version>` из `/api/system-info`. |
| 6 | **Модели шумоподавителя** | Находит модели `*.pth.enc` (в Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + фильтр шума**| Функция Texture Aware действительно работает — требуется CUDA**и** как минимум один файл модели. |

Запуск завершается с `N/7 checks passed`, с перечнем всех сбоев по названиям.

### 4. Обработка вашего первого набора данных

```bash
chloros-cli process ~/datasets/flight001
```

***

## Файлы и каталоги

### Для каждого пользователя

Chloros хранит свои учетные данные и конфигурацию CLI в одном кроссплатформенном каталоге **`~/.chloros/`** (в Windows — `%USERPROFILE%\.chloros\`). Два кэша, специфичных для Linux, вместо этого следуют конвенциям XDG — при настройке они используют имена `XDG_CONFIG_HOME` / `XDG_CACHE_HOME`.

| Путь | Назначение |
| --- | --- |
| `~/.chloros/user_session.json` | Кэш сеанса входа, записываемый `chloros-cli login` (сохраняется при обновлении пакета; очищается при удалении пакета) |
| `~/.chloros/working_directory.txt` | Переопределение папки проекта по умолчанию (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Настройка языка интерфейса CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Настройка языка, общая с графическим интерфейсом Windows — значение `language` здесь имеет приоритет над `cli_language.json` |
| `~/.chloros/update_cache.json` | Одночасовой кэш для проверки обновлений при запуске Linux/Jetson |
| `~/.chloros/backend.log` | Журнал бэкэнда при запуске бэкэнда через CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Кэшированные пакеты калибровки LATTICE для каждой камеры, индексированные по серийному номеру и хешу пакета |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Дополнительные пользовательские настройки для профилей коррекции капа DAQ |
| `~/.config/chloros/system_config.json` | Кэшированный профиль оборудования из Dynamic Compute Adaptation — удалите его, чтобы принудительно запустить новое обнаружение оборудования |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Журналы бэкэнд-сервера, по одному файлу на каждый запуск |
| `~/Documents/MAPIR/Chloros Projects/` | Папка проекта по умолчанию, если не задано никаких настроек |

### Системные

| Путь | Назначение |
| --- | --- |
| `/usr/bin/chloros-cli` | Скрипт-обёртка — устанавливает `LD_LIBRARY_PATH` для входящих в комплект нативных библиотек (а также `TMPDIR` в `/mnt/ssd/tmp`, если такая папка существует), после чего запускает собственно бинарный файл |
| `/usr/bin/chloros-backend` | Скрипт-обёртка — то же самое для бэкэнда, который он помечает как установленную производственную сборку |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | Скомпилированные версии CLI и бэкенда, устанавливаемые в распакованном виде (новое в версии 1.3.0); программы называются `cli/chloros-cli` и `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Двустрочные скрипты-запуска, которые запускают указанные выше программы — хранятся в этих путях, поскольку SDK, CLI и старые скрипты ищут их именно здесь |
| `/usr/lib/chloros/arena_runtime/` | Среда выполнения Arena SDK, необходимая для камер LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Зашифрованные модели шумоподавителя, используемые дебайером Texture Aware |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (а также `lattice_sdk`, `lattice_cam`, `arena_api` — загрузчики проекта) | Python SDK, установленный для системного модуля `python3` и принадлежащий данному пакету |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Колесо Python SDK, соответствующее именно этой сборке, для virtualenv |
| `/usr/lib/chloros/exiftool` | Входящий в комплект запускатель ExifTool (создаёт символьную ссылку на `/usr/local/bin/exiftool` только в том случае, если системный exiftool отсутствует). Начиная с версии 1.3.0 скрипт ExifTool поставляется со своей собственной библиотекой Perl в `/usr/lib/chloros/exiftool-dist/`, поэтому он работает без системного пакета `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Конфигурация канала обновлений, считываемая `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Устанавливает `net.ipv4.ip_unprivileged_port_start = 319`, чтобы бэкенд мог привязывать порты PTP без прав root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Увеличивает буферы приёма сокетов ядра (`net.core.rmem_max`, `net.core.rmem_default`) до 32 МБ для потоков с камер LATTICE и применяет эти настройки при установке (если значение уже установлено выше, оно остаётся без изменений). При настройках ядра по умолчанию пакеты с камер отбрасываются в ядре, и массив теряет кадры и частоту кадров; `chloros-cli lattice network` сообщает о низком уровне буферов. См. [Хосты Linux — буферы приёма сокетов](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Направляет динамический загрузчик на `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Предоставляет авторизованному пользователю доступ к USB-мосту DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Включаемая служба «always-on», запускающая бэкенд от имени указанного вами пользователя (установлена, **не включена**) — именно её нужно включить, см. [Always-On PTP](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | Та же служба, запускающая бэкенд от имени **root**(установлена,**не включена**), сохраняется для того, чтобы машина, на которой она включена, продолжала работать |
| `/var/log/chloros/` | Подробный журнал службы root, доступный для чтения группе `adm` (удаляется при полном удалении пакета) |
| `/usr/share/applications/chloros-cli.desktop` | Пункт меню приложения «Chloros CLI», открывающий терминал |

**Брандмауэр (новое в версии 1.3.0).** Если на момент установки активны `ufw` или `firewalld`, пакет открывает UDP-порты `5002` (данные DAQ-E), `5353` (обнаружение mDNS), `319` и `320` (PTP), а при удалении пакета эти правила удаляются. Если брандмауэр установлен, но не активен, он ничего не добавляет и выводит список портов, которые необходимо разрешить, если вы включите его позже. Он не открывает никаких портов TCP: обновление прошивки DAQ-E, при котором датчик подключается обратно к этому компьютеру по TCP для сбора изображения, завершается с ошибкой `… device never connected back on TCP <port> …`, если активный брандмауэр блокирует это соединение.

## Расположение исполняемого файла бэкэнда

CLI и SDK автоматически определяют бэкэнд:

| Компонент | Путь |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Бэкенд | `/usr/lib/chloros/chloros-backend` (программа запуска для `/usr/lib/chloros/backend/chloros-backend`) |

Переопределите путь к бэкенду с помощью флага CLI `--backend-exe` или параметра конструктора SDK `backend_exe`, а порт — с помощью `--port` (по умолчанию `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` указывает на **`lattice`**,**`project`**и**`daq pool-*`** на удалённый бэкенд. Основные команды (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) намеренно игнорируют его и всегда обращаются к `http://127.0.0.1:<port>`.
{% endhint %}

***

## Камеры LATTICE и световые датчики DAQ в Linux

Все семейства команд live-hardware работают в Linux (amd64 и Jetson):

* **`chloros-cli lattice`** — обнаружение, подключение, настройка и захват данных с камер LATTICE и синхронизированных массивов. Команда `.deb` объединяет необходимую им среду выполнения Arena SDK и регистрирует её в динамическом загрузчике.
* **`chloros-cli daq pool-*`** — подключение световых датчиков DAQ-U/M/E через пул бэкэндов, потоковая передача откалиброванных спектров, запись файлов `.daq` и обмен профилями капсул, а также, для DAQ-E, изменение его сетевых настроек, перезапуск и обновление прошивки. `chloros-cli daq discover` сканирует датчики, к которым можно подключиться, а `chloros-cli daq --help` выводит список подкоманд `pool-*`.
* **`chloros-cli project`** — запускает сохраненный проект (его камеры, датчики и настройки обработки) в режиме без интерфейса.
* **`chloros-cli time-sync`** — проверка PTP-грандмастера, на котором работает бэкенд Chloros для камер LATTICE и датчиков DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

`--sensor-id` является обязательным условием для выполнения команд `pool-latest`, `pool-stream`, `pool-record` и `pool-set-cap`; `pool-list` отображает идентификаторы, находящиеся в данный момент в пуле.

{% hint style="info" %}
**`--eth-host` пропускает обнаружение.** `pool-connect --eth` находит DAQ-E с помощью mDNS, с резервным ARP, который работает из «холодного» кэша ARP; передача IP-адреса или имени хоста датчика с помощью `--eth-host` позволяет подключиться к нему напрямую.
{% endhint %}

**Права доступа к последовательному порту DAQ-U** регулируются установленным правилом udev (`uaccess` + группа `dialout`). Если датчик, который уже был подключен, остаётся недоступным, перезагрузите правила или подключите его заново:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Полный набор команд см. в [справочнике по CLI](../CLI.md).

### Постоянно включенный PTP для хостов без монитора

Пакет устанавливает две службы systemd для бэкэнда, ни одна из которых не включена. На безмониторном Jetson или сервере, где должна постоянно работать синхронизация времени PTP для датчиков DAQ-E и камер LATTICE, либо там, где скрипты SDK подключают сохраненные проекты (`connect_all()` никогда не запускает бэкенд), включите службу **от имени вашего пользователя**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

`chloros-backend@<user>.service` запускает бэкенд от имени этого пользователя: записываемые им данные принадлежат этому пользователю, его подробный журнал находится в файле `~/.cache/chloros/logs`, и он использует учетную запись `chloros-cli login` (`~/.chloros`), поэтому сначала войдите в систему один раз под этим именем пользователя. При установке пакета выводится эта команда с подставленным вашим именем пользователя.

Без службы бэкенд (а вместе с ним и PTP-грандмастер) работает только тогда, когда его запускает что-либо — команда `chloros-cli` или скрипт SDK.

Модуль привязывает бэкенд к настройкам среды `127.0.0.1:5000` (настройки среды `CHLOROS_HOST` / `CHLOROS_PORT` внутри модуля; переопределяются с помощью `sudo systemctl edit chloros-backend@.service`) и перезапускает его в случае сбоя через 5 секунд.

**Служба root.**`chloros-backend.service` запускает бэкенд от имени**root**. Она по-прежнему установлена, чтобы машина, на которой она включена, продолжала работать, но всё, что записывает бэкенд с правами root, принадлежит root, поэтому пользователь не может удалить свои собственные записи; его журнал находится в `/var/log/chloros` и доступен для чтения группе `adm`. Чтобы переключиться на своего пользователя:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**Бэкенд службы является общим и никогда не останавливается клиентом.** Скрипты `chloros-cli` и SDK используют его в том виде, в каком он есть: `process` запускается на нём, загружая свой собственный проект вместо последнего. В тех случаях, когда CLI, в противном случае, перезапустил бы бэкенд (`--restart` или `process`, импорт которого ничего не нашёл), он выводит название модуля, печатает `sudo systemctl restart <unit>` и завершает работу `1`, поскольку бэкенд, остановленный таким образом, остался бы в нерабочем состоянии. Перезапустите его самостоятельно с помощью `sudo systemctl restart chloros-backend@$USER.service`. Бэкенд службы никогда не останавливается самостоятельно в режиме простоя, и команда `chloros-cli backend stop` отклоняет такую попытку, называя команду `systemctl stop`.**Как PTP получает свои порты.** PTP использует UDP 319/320, оба из которых находятся ниже обычного нижнего предела привилегированных портов — 1024. `postinst` пакета записывает `/etc/sysctl.d/60-chloros-ptp.conf` с помощью `net.ipv4.ip_unprivileged_port_start = 319`, что позволяет бэкенду привязывать их при работе от имени вашего пользователя. Начиная с версии 1.3.0 пакет не устанавливает никаких файловых возможностей (`setcap`) на бэкенде, а `libcap2-bin` больше не является зависимостью: на распакованном бэкенде файловая возможность помешала бы его запуску, поэтому не добавляйте его самостоятельно.

***

## Примеры скриптов на Bash

{% hint style="info" %}
**Коды завершения, удобные для скриптов.**`chloros-cli process` завершает работу с кодом `0` при успешном выполнении и**с ненулевым кодом при сбое — включая запуск, в ходе которого запрашивались продукты изображений, но ни один из них не был записан** (он выводит `Processing finished but wrote no image products.` и указывает имя папки проекта, а также обычные причины). При успешном выполнении сообщается, сколько изображений было записано (`Image products written: N`, плюс `(plus M clamp masks)`, если были записаны маски). Коды завершения: `0` — успех, `1` — сбой, `2` — ошибка аргумента, `130` — прерывание.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Допустимых значений `--format` ровно четыре, и они содержат пробелы — всегда заключайте их в кавычки:

| Значение `--format` | Папка вывода |
| --- | --- |
| Значение `TIFF (16-bit)` *(по умолчанию)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` допускает значения `standard` (по умолчанию) или `texture-aware` (Chloros+).

### Автоматическая обработка с помощью Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Пример использования SDK-Python для Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### Бэкенд не запускается после добавления прав доступа к файлам

Не запускайте `setcap` на бэкенде. Начиная с версии 1.3.0 пакет не устанавливает никаких файловых возможностей, а порты PTP 319/320 задаются настройкой `sysctl`, описанной в разделе [Как PTP получает свои порты](#always-on-ptp-for-headless-hosts). Файловая способность переводит распакованный бэкенд в режим безопасного выполнения, в котором он не может найти свои собственные библиотеки, поэтому не запускается. Если вы добавили такую способность, удалите её:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
```

### PTP не запускается / не может привязаться к порту 319

Убедитесь, что нижний порог непривилегированных портов был понижен, и, если это не было сделано, примените его заново для текущей загрузки:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Затем проверьте главный сервер:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### Команды `lattice` завершаются с ошибкой «Не найдены драйверы камеры LATTICE»

Бэкенд не смог загрузить среду выполнения камеры LATTICE, с которой он был установлен. Каждая команда `lattice` выполняется на бэкенде; сам интерфейс CLI не содержит драйверов камеры. Переустановите Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): установка `.deb` устанавливает среду выполнения рядом с бэкендом, в `/usr/lib/chloros/arena_runtime`, и регистрирует её в динамическом загрузчике. Чтобы проверить эту регистрацию:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
ls /usr/lib/chloros/arena_runtime | head
```

### Сбой запуска бэкэнда

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

Журналы бэкенда, связанные с неудачным запуском, находятся в файле `~/.cache/chloros/logs/`. Если порт 5000 принадлежит службе бэкенда (`systemctl status chloros-backend@$USER.service`), перезапустите службу, а не завершайте её процесс.

### CUDA не обнаружена

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

`chloros-cli selftest` сообщает об этом же в одной строке: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Отсутствуют общие библиотеки

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Медленный запуск на системах с SD-картами

Начиная с версии 1.3.0 бэкэнд и интерфейс командной строки (CLI) устанавливаются в распакованном виде, поэтому они больше не распаковываются во временно папку при каждом запуске. Временные файлы автоматически сохраняются в папке `/mnt/ssd/tmp`, если она существует; в противном случае вы можете указать для них быструю файловую систему:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Обновление Chloros на Linux

Команда `update` доступна только для Linux/Jetson. Она проверяет версию, опубликованную в канале обновлений, настроенном в `/etc/chloros/update.conf`, и предлагает загрузить и установить соответствующий файл `.deb`:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

В Linux/Jetson интерфейс CLI также выполняет неблокирующую проверку обновлений при каждом запуске (результат кэшируется в течение одного часа в `~/.chloros/update_cache.json`) и выводит сообщение `Update available: vX.Y.Z` при наличии более новой версии. Ваши настройки, проекты и данные входа сохраняются после обновления.

Обновление пакета приостанавливает работу всех бэкенд-сервисов на время установки, поэтому запущенный скрипт теряет доступ к камерам и датчикам, а затем запускает заново каждый из них, который включён (сервис, запущенный вручную, но никогда не включённый, остаётся остановленным, как после перезагрузки).

## Удаление

```bash
sudo apt remove chloros
```

Удаление останавливает и отключает все фоновые службы, восстанавливает по умолчанию нижний порог портов без привилегий (1024), удаляет файл настроек буфера приёма (текущие значения сохраняются до перезагрузки), символьную ссылку на входящий в комплект exiftool, конфигурацию загрузчика Arena, правила брандмауэра, добавленные пакетом, и установленный им Python-SDK, а также очищает кэшированные учетные данные. Ваши проекты и файлы данных `~/.chloros/` остаются нетронутыми. `sudo apt purge chloros` также удаляет `/var/log/chloros`.

***

## Следующие шаги

* [Руководство по NVIDIA Jetson](nvidia-jetson-guide.md) — Оптимизация и развёртывание специально для Jetson
* [CLI: командная строка](../CLI.md) — руководство по CLI
* [API: Python SDK](../api-python-sdk.md) — руководство по SDK
* [Справочник по CLI](../reference/cli-reference.md) и [Справочник по SDK](../reference/sdk-reference.md) — исчерпывающие списки команд и API для версии 1.3.0
* [Динамическая адаптация вычислительных ресурсов](../processing-architecture/dynamic-compute-adaptation.md) — как Chloros адаптируется к вашему оборудованию
