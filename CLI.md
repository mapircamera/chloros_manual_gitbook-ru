# CLI : Командная строка

> **Полное руководство:**[CLI Справочник](reference/cli-reference.md) содержит информацию о**всех параметрах всех подкоманд** и оптимизирован для ИИ-помощников — вставьте его URL в свой помощник и запросите рабочую команду: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Совет для ИИ-инструментов:** любая страница этого руководства доступна в виде исходного кода Markdown, если добавить `.md` к её URL-адресу URL (например, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), а `https://mapir.gitbook.io/chloros/llms.txt` индексирует всё руководство для использования LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## Что такое «CLI
»

`chloros-cli` — это интерфейс командной строки для того же механизма обработки, который используется в настольном приложенииChloros
. Это «тонкий» клиентHTTP
, работающий через бэкендChloros
(локальный сервер на `127.0.0.1:5000`) — большинство команд запускают бэкенд автоматически, поэтому для скрипта достаточно одного вызова `chloros-cli process …`.

Оно работает на **Windows
10/11 (x64)**и**Linux
(x86_64, а также NVIDIA Jetson arm64 на JetPack 6)**, в любом терминале, без необходимости использования графического интерфейса. Проверьте установку с помощью команды:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Краткий обзор семейств команд:

* **Обработка и учёт** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 языков — см. [Поддерживаемые языки](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (только дляLinux
/Jetson)
* **Рабочее оборудование** — `lattice` (управление камерой LATTICE, более 45 подкоманд), `daq pool-*` (датчики освещенности DAQ), `time-sync` (PTP)
* **Автоматизация** — `project` (запуск сохраненного проектаChloros
в режиме без интерфейса, включая рецепты съемки в формате YAML)

Глобальные параметры, о которых стоит знать: `--port N` (порт бэкенда, по умолчанию `5000`), `-v/--verbose`, `--restart` (принудительный перезапуск бэкенда), `--backend-exe PATH`. Полный список см. в [Справочнике поCLI
](reference/cli-reference.md).

***

## Установка

CLI
**входит в состав установщикаChloros** для всех платформ — отдельной загрузкиCLI
нет. Загрузите установщик со страницы [«Загрузка»](download.md).

###Windows


Установщик помещает файлCLI
в папку:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

и добавляет эту папку в системный путь `PATH` — **откройте новый терминал**после установки, чтобы система обнаружила обновлённый файл `PATH`. Установщик также помещает скрипты запуска (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) в корневую папку установки, а также**Chloros
CLI
** ярлык в меню «Пуск», каждый из которых открывает терминал с готовым к использованию `chloros-cli`.

###Linux


Установите `.deb` для вашей архитектуры:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Это установит версии от `chloros-cli` до `/usr/bin/chloros-cli` (уже установлена версия `PATH`), а также бэкенд до версии `/usr/lib/chloros/chloros-backend`, вместе с средой выполнения ArenaSDK
, необходимой для камер LATTICE. Подробности см. в разделе [УстановкаLinux
](linux/linux-installation.md).

### Проверка

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Вход в систему и лицензирование

CLI
(а такжеPython
SDK
) доступ требует **платного тарифного планаChloros
+**— он доступен в любом платном тарифном плане; в бесплатном тарифном плане его нет. Ограничение на вход в систему обеспечивается**на стороне сервера** бэкендом, а не бинарным файломCLI
: вызов без входа в систему отклоняется с кодом ошибки `401 AUTH_REQUIRED`, а вызов, выполненный пользователем, вошедшим в систему на бесплатном тарифе, — с кодом `403 PLAN_UPGRADE_REQUIRED`, независимо от того, поступает ли он из `chloros-cli`,SDK
или из самостоятельно разработанного клиентаHTTP
. Обновление доступно по адресу [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Войдите в систему **один раз с каждого компьютера**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Пароли со специальными символами**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` искажается оболочкой (CLI
обнаруживает это по коду 401 и автоматически повторяет попытку, но использование одинарных кавычек позволяет полностью избежать этой проблемы).
{% endhint %}

Сессия кэшируется в `~/.chloros/user_session.json` и продолжает работать в автономном режиме в течение льготного периода тарифного плана (30 дней для месячных тарифных планов, до истечения срока действия для годовых). `chloros-cli status` работает даже без платного тарифного плана, поэтому причина отказа всегда видна.

{% hint style="danger" %}
**Планируете запуск задач в фоновом режиме? Сначала войдите в систему.**Команды, запускающие бэкэнд (`process`, `status`, `export-status`, …) при запуске**без кэшированной сессии**не завершается с ошибкой сразу — она переходит в интерактивный режим с приглашением `Email:` / `Password:` на stdin. Поэтому автозапускаемое задание cron или этап CI**зависнет в ожидании ввода**. Перед планированием каких-либо задач запустите `chloros-cli login EMAIL 'PASSWORD'` один раз на компьютере.
{% endhint %}

***

## Ваш первый запуск обработки

Укажите `process` на папку с записями — он автоматически обнаружитSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` или их комбинацию:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Поток прогресса отображается в реальном времени для каждого потока конвейера (обнаружение, анализ, обработка, экспорт), и успешное завершение запуска сопровождается отчётом о количестве записанных изображений (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Куда сохраняются результаты

`process` записывает данные в **папку проекта**, а не в вашу входную папку:

* Если не указан параметр `-o`: проект создаётся в папке проектов по умолчанию (общей с графическим интерфейсом; управление осуществляется с помощью `get-project-folder` / `set-project-folder`, резервный вариант — `~/Chloros Projects`), и получает имя, заданное параметром `-n/--project-name` или временной меткой (`YYYYMMDD_HHMMSS`), если этот параметр не указан.
* С `-o PATH`: эта папка **является** папкой проекта. Если она уже содержит файл `project.json`, вместо его перезаписи создаётся файл с суффиксом `_1`/`_2`…

Внутри проекта продукты сгруппированы **по камере, а затем по формату файла**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

Папка камеры — `LATT-<sensor>-<lens>-F<filter>` для LATTICE (соответствует EXIF-данным съемки `Model`) и `<model>_<filter>` (например, `Survey3N_RGN`) для камерыSurvey3
. Папка формата следует за `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` или `tiff32` для `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Каждый экспортированный продукт сохраняет имя исходного файла.**Экспорт радианса из `capture_..._raw.tif` по-прежнему называется `capture_..._raw.tif` — он просто находится в папке `tiff32/Radiance_Images/`.**Продукт идентифицируется по папке, а не по имени файла**, поэтому используйте шаблон для поиска каталога, а не для суффикса `*radiance*`.
{% endhint %}

### Параметры, которые вы будете использовать на практике

| Флаг | По умолчанию | Что он делает |
| --- | --- | --- |
| `-o, --output PATH` | папка проекта по умолчанию | Расположение папки проекта (см. выше). |
| `-n, --project-name NAME` | метка времени | Название проекта. |
| `--format FMT` | `TIFF (16-bit)` | Один из следующих: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | нет | Индексы растительности для экспорта (см. [Индексы растительности](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = нейронный дебейер, более медленный, наивысшее качество (Chloros
и выше, графический процессор NVIDIA). |
| `--vignette / --no-vignette` | включено | Коррекция виньетирования. |
| `--reflectance / --no-reflectance` | включено | Калибровка отражательной способности; для LATTICE это также переключатель продукта отражательной способности. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Принудительное задание точки входа в конвейер для TIFF-файлов LATTICE. |

По всем остальным параметрам — настройка обнаружения целей, PPK, фиксаторы экспозиции, флаги выравнивания массивов — см. [раздел `process` в справочнике поCLI
](reference/cli-reference.md).

***

## Выбор элементов для экспорта (продукты LATTICE)

Обработка LATTICE распределяется **по всем подходящим продуктам за один проход**. Четыре переключателя для каждого продукта**по умолчанию включены**; используйте форму `--no-`, чтобы отключить один из них:

| Переключатель | Продукт |
| --- | --- |
| `--debayered` | Линейная демозаика → `Debayered_Images/` |
| `--preview` | Предварительный просмотр (баланс белого + гамма; растяжение ложных цветов для мультиспектральных изображений) → `Preview_Images/` |
| `--radiance` | яркость (float32), Вт/м²/sr/нм → `Radiance_Images/` (всегда `tiff32/`) |
| `--reflectance` | uint16 коэффициент отражения, готовый для Pix4D → `Reflectance_Calibrated_Images/` |

RGB
главные камеры всегда выдают только данные после дебайеризации + предварительный просмотр — радиантность/отражательная способность по отдельным полосам не имеют смысла для широкополосного датчика, поэтому эти переключатели для них не действуют.Survey3
`.raw` игнорирует переключатели и следует стандартному пути отражательной способности/целевому пути.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (по умолчанию `auto`) выбирает эталон отражательной способности: `auto` создаёт прошедшую контроль качества [калибровочную мишень](calibration-targets.md) в кадре в качестве абсолютного эталона и переключается на значение деления нисходящего излучения светового датчика DAQ (ρ = π·L/E) при отсутствии мишени; `target` действует строго (без подстановки данных DAQ); `daq` полагается на данные DAQ. Сканированные данные по измерённым мишеням в единицах измерения могут быть предоставлены с помощью `--target-reflectance-dir`.

{% hint style="info" %}
**Считывание пикселей отражательной способности:**значение DN, соответствующее ρ = 1,0, является**для каждого источника** — Файлы LATTICE проставляют тег `Chloros:PixelScale=32768` в XMP; файлыSurvey3
используют значение 65535 (и не содержат тегов `Chloros:*`). Считывайте тег и делите на его значение, а не принимайте за постоянную. Подробности и единственный преднамеренный крайний случай без масштабирования приведены в [Справочнике поCLI
](reference/cli-reference.md).
{% endhint %}

**Обработка всегда начинается с `raw`.** Производные продукты (экспорт данных после дебайеризации, яркости и отражательной способности) никогда не возвращаются в конвейер — их повторный импорт и обработка привели бы к двойному применению калибровочных вычислений, поэтомуChloros
пропускает их и сообщает об этом. `--input-level` — это специально предусмотренный «запасной выход» на случай, если вам действительно нужно принудительно задать точку входа.

***

## Когда запуск завершается сбоем

Начиная с версии 1.2.0, `process` явно завершает работу с ошибкой, а не «успешно» без вывода результатов:

* Запуск, который **запрашивал продукты, но не записал ни одного**— только `project.json` и `calibration_data.json` — выводит `Processing finished but wrote no image products.` и**завершается с ненулевым кодом**, поэтому скрипты могут это обнаружить. Обычные причины: входная папка не была распознана как захват (проверьте структуру и `--input-level`), или все запрошенные продукты были неприменимы для этих камер (например, запрос на яркость/отражательную способность от камер, поддерживающих только форматRGB
).
* **Намеренный запуск только с метаданными** (все продукты отключены, без `--indices`) всё равно считается успешным — пустой выходной файл изображения в данном случае является правильным результатом.
* Запустите процесс заново с параметром `--verbose` и проверьте журнал бэкэнда на наличие строк `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, которые объясняют пропуски по камерам.

Коды завершения: `0` — успех · `1` — общая ошибка · `2` — ошибка аргумента · `130` — прервано нажатием Ctrl+C.

***

## Индексы растительности

Запустите `--indices` с одним или несколькими именами предустановок; каждый индекс помещается в свою собственную папку `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

22 названия предустановок, которые принимает `process --indices`:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Существует три списка индексов — не путайте их.**В раскрывающемся списке «Настройки проекта» графического интерфейса пользователя имеется 27 формул (добавлены `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — эти пять доступны только в графическом интерфейсе и**не** действительны для `--indices`). Команда «в режиме реального времени/в автономном режиме» `lattice index --preset` использует свой собственный отдельный список из 22 предустановок. Формулы и математические вычисления по полосам описаны в разделе [Формулы мультиспектральных индексов](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Световые датчики DAQ: краткий обзор

Семейство `daq pool-*` управляет спектральными датчиками DAQMAPIR
(DAQ-U по USB, DAQ-M по BLE, DAQ-E по Ethernet) через постоянный пул бэкэнда — графический интерфейс пользователя,CLI
иSDK
используют один общий дескриптор для работы в режиме реального времени. **`pool-*` — это поддерживаемый путь DAQ в поставляемом пакете «CLI
»**; другие подкоманды `daq`, на которые вы можете встретить ссылки, представляют собой внутренний интерфейсMAPIR
, предназначенный только для источников, и завершают работу с явной ошибкой, указывающей на `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` без `--duration` работает до `pool-record --stop`; каталог вывода по умолчанию — `~/Documents/DAQ Live View/` **на компьютере бэкенда**. Профиль коррекции емкости выбирается при подключении (`--cap-id`, значение по умолчанию бэкенда — `sunshine_cosine`) и может быть заменен в режиме реального времени с помощью `pool-set-cap` — профили капа и откалиброванный диапазон датчика описаны в главах, посвящённых DAQ, данного руководства.

{% hint style="warning" %}
**DAQ-E на хосте с несколькими сетевыми картами:** первое автоматическое обнаружение `pool-connect --eth` после загрузки может завершиться сбоем даже при исправном датчике. `--eth-host <ip-or-hostname>` — это надёжный вариант; используйте его всегда, когда обнаружение не даёт результатов.
{% endhint %}

***

## Камеры LATTICE, PTP и автоматизация проектов

Семейство `lattice` (более 45 подкоманд) охватывает полный цикл работы с камерами LATTICE: обнаружение, единичные снимки, постоянные синхронизированные массивы с помощью интеллектуального процесса подключения в графическом интерфейсе, предварительный просмотр в браузере в режиме реального времени, выравнивание, индексные вычисления и диагностика сетевых карт хоста. Пример:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Наряду с этим: `chloros-cli time-sync` предоставляет отчет о «главном» устройстве PTP, запущенном на хостеChloros
(камеры LATTICE и датчики DAQ-E работают в режиме «ведомого» для получения межустройственных временных меток), а `chloros-cli project` открывает сохраненный проектChloros
и управляет его камерами, массивами и датчиками в режиме без графического интерфейса — включая скриптовые рецепты сбора данных в формате YAML.

Эти три семейства (`lattice`, `project`, `daq pool-*`) также являются единственными, которые поддерживают использование `CHLOROS_BACKEND_URL` для управления **удалённым** бэкендом; основные команды всегда ориентированы на локальную машину.

Полные пошаговые инструкции приведены в главах, посвящённых LATTICE, данного руководства; все флаги описаны в [Справочнике поCLI
](reference/cli-reference.md).

***

## Устранение неисправностей: 5 основных проблем

| Симптом | Решение |
| --- | --- |
| `Login required` или запланированная задача зависает на приглашении `Email:` | Запустите `chloros-cli login EMAIL 'PASSWORD'` один раз на этом компьютере — команды без кэшированной сессии будут выполняться в интерактивном режиме, а не завершаться с ошибкой. |
| `backend unreachable` | Запустите настольное приложениеChloros
или запустите бинарный файл бэкэнда напрямую (`chloros-backend`). Если вы указываете `lattice`/`project`/`daq pool-*` на удалённый бэкенд, проверьте `CHLOROS_BACKEND_URL`. |
| Блокировка подключения к массиву: `FRAMES WILL DROP` / `Reduce ROI to enable` | Сброс настроек кольца приёма сетевой карты хоста до значений по умолчанию — основная причина отказа ранее работавшей установки от подключения, как правило, после обновления драйвера сетевой карты. Запустите `chloros-cli lattice network --fix` из терминала с **повышенными привилегиями** (или установите `ReceiveBufferLen=256`, `PendingReceives=64`); см. раздел *Настройка и оптимизация сетевой карты хоста* в справочнике. |
| Подкоманда `daq` завершает работу с сообщением: «требуется полный пакет daq…» | Ожидаемо для поставляемых сборок — скомпилированный пакетCLI
поставляется только с семейством команд `daq pool-*`, которое охватывает подключение, потоковую передачу, запись и выбор капсулы. Используйте `pool-*` (или `chloros_sdk.connect_daq_sensor()` изPython
). |
| Jetson выводит предупреждение о подкачке перед обработкой больших папок | Добавьте подкачку на основе файлов — файлCLI
выводит точные команды `fallocate`/`swapon` для запуска. |

***

## Получение справки

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Все флаги, все подкоманды:** [CLI
Справочник](reference/cli-reference.md)
* **ЭквивалентPython
:** [Python
SDK
](api-python-sdk.md) и [SDK
Справочник](reference/sdk-reference.md)
* **Поддержка:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
