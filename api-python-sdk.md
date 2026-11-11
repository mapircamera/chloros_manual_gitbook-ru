# API: Python SDK

{% hint style="info" %}
**Ищете полную документацию по API?** Эта страница представляет собой практическое руководство. Все общедоступные классы, методы, точные сигнатуры и примеры, готовые к копированию и вставке, находятся в [справочнике по SDK](reference/sdk-reference.md), который оптимизирован для работы с AI-помощниками.**Работаете с ИИ-помощником?** Вставьте этот URL в чат, чтобы получить полную актуальную версию API Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Каждая страница этого руководства доступна в виде исходного кода Markdown по адресу, состоящему из строчного слэга + `.md`, а всё руководство индексировано по адресу `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` на PyPI) обеспечивает управление всеми функциями настольного приложения из Python: пакетная обработка изображений, управление камерой LATTICE и массивом в режиме реального времени, сеансы сбора данных с световых датчиков, а также автоматизация сохраненных проектов. Это тонкий слой, наложенный на тот же локальный бэкенд, который используют графический интерфейс и интерфейс командной строки (HTTP на `127.0.0.1:5000`), поэтому поведение одинаково во всех трёх средах.

## Установка

SDK поставляется в составе пакета Chloros для настольных компьютеров, который также содержит бэкэнд обработки и среды выполнения для аппаратного обеспечения, и его версия всегда совпадает с версией пакета.

**Шаг 1 — Установите Chloros.** Windows: запустите установщик для настольных компьютеров (путь по умолчанию — `C:\Program Files\Chloros\`) со страницы [Скачать](download.md). Linux: установите пакет `.deb` с помощью `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Установка в Linux](linux/linux-installation.md)).**Шаг 2 — Проверка SDK** (Python 3.7+). Обычно ничего устанавливать не нужно:

* **Windows:** установщик устанавливает входящий в комплект пакет SDK wheel в системный Python (если Python не установлен, этот шаг пропускается).
* **Linux:** `.deb` устанавливает SDK для системного `python3` в `/usr/lib/python3/dist-packages`. Требования к системе, `python3-numpy` и `python3-requests`, рекомендуются пакетом: `apt install` включает их, а `dpkg -i` — нет.
* **Виртуальная среда** не видит системные пакеты; установите в неё входящий в комплект файл wheel: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` в Linux или файл `chloros_sdk-*.whl` из папки `sdk` в папке установки Windows (по умолчанию — `C:\Program Files\Chloros\sdk\`).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

На компьютере, на котором отсутствует пакет Chloros, чьё SDK управляет бэкендом на другом компьютере (см. [Что получает установка только с помощью pip](#what-a-pip-only-install-gets)), установите его с PyPI: `pip install chloros-sdk`. PyPI обновляется при выпуске новых сборок, поэтому он соответствует последней стабильной версии.

**Шаг 3 — Войдите в систему один раз на каждом компьютере:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Учетные данные кэшируются в `~/.chloros/` (на обеих платформах). В Windows вы можете аналогичным образом войти через вкладку «Пользователь» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> в настольном приложении. Для работы SDK требуется платный тарифный план Chloros+ — см. [Требования к лицензии](#license-requirement) ниже.

| Требование | Подробности |
| --- | --- |
| **Установлен Chloros** | Windows: установщик для настольных компьютеров; Linux: пакет `.deb` (содержит бинарный файл бэкэнда и SDK) |
| **Язык Python** | 3.7 или выше (разработано/протестировано на версии 3.10) |
| **Операционная система** | Windows 10/11 (64-разрядная версия), Ubuntu 22.04 LTS или более поздние версии, либо NVIDIA Jetson (JetPack 6) |
| **Лицензия** | Активная учётная запись Chloros+, любой платный тариф (Copper или выше) |

## Результат за 60 секунд

Один вызов создаёт проект, импортирует папку, настраивает обработку и запускает конвейер — при этом бэкенд запускается автоматически, если он ещё не работает:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(В Linux используйте пути Linux: `/home/user/drone_images/flight001`. SDK работает одинаково на обеих платформах.)

Обрабатываете папку с данными LATTICE? Используйте оболочку, оптимизированную для LATTICE — она применяет правильные настройки по умолчанию (без определения целевой панели, стандартный дебейер):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — полное управление конвейером

Для любых задач, выходящих за рамки однострочной команды, используйте `ChlorosLocal`. Он использует уже запущенный бэкэнд (настольного приложения) или запускает установленный при первом использовании (`auto_start_backend=True`; в Windows — из папки `backend` в папке установки Chloros, по умолчанию — `C:\Program Files\Chloros\backend\`), создаёт и настраивает проекты, отслеживает ход выполнения и возвращает сводку по итогам выполнения.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
Сохраните значение по умолчанию `http://127.0.0.1:5000`, а не заменяйте его на `localhost` — в Windows `localhost` сначала преобразуется в `::1`, что занимает около 2 секунд на каждый запрос при работе с бэкендом, поддерживающим только IPv4.
{% endhint %}

Используйте его в качестве контекстного менеджера. Выход из блока прекращает использование бэкэнда данным экземпляром, но не сам бэкэнд: он продолжает работать для следующего скрипта и останавливается самостоятельно после 30 минут простоя. `shutdown_backend()` останавливает один запущенный данным экземпляром.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` поддерживает следующие ключевые слова: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` и `custom_settings`. Основные значения:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Параметры, специфичные для LATTICE (`input_level`, `radiometric_output`, семейство `array_alignment*`), описаны с полными таблицами значений в [Справочнике по SDK](reference/sdk-reference.md#supported-values).

### Отслеживание хода выполнения

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Чтение сводки по итогам выполнения — и обнаружение пустых прогонов

По завершении `process()` присоединяет сводку обработки бэкенда в виде `result["summary"]`. Каждая запись в `summary["hints"]` представляет собой полное предложение, объясняющее всё, что заслуживает внимания — например, почему запуск не дал никаких результатов — и каждая подсказка также повторно выводится в виде Python-кода `UserWarning`, поэтому пустые запуски диагностируются автоматически, даже если вы никогда не просматриваете словарь:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` не генерирует исключение, когда запуск не создаёт изображений.** Это единственное место, где SDK и CLI намеренно различаются: `chloros-cli process` рассматривает ситуацию «продукты были запрошены, но ни один не был записан» как сбой и завершается с ненулевым кодом, тогда как SDK возвращается в нормальном режиме и сообщает об этом состоянии через `summary` / подсказки. Если ваш конвейер должен останавливаться при пустом запуске, проверяйте это самостоятельно — анализируйте `summary` (или подсчитывайте файлы в папке проекта), а не полагайтесь на исключение.
{% endhint %}

## Smart Connect — аппаратное обеспечение в режиме реального времени

Три вспомогательных процесса открывают постоянные сессии в пуле аппаратного обеспечения бэкэнда — том же пуле, который используют CLI, другие скрипты и вкладки камер в графическом интерфейсе, поэтому скрипты SDK сосуществуют с ними, не конкурируя за последовательные порты или пропускную способность сети. Все три автоматически запускают локальный бэкэнд, если он не запущен. Панель DAQ графического интерфейса не входит в этот пул: закройте датчик освещенности в графическом интерфейсе, прежде чем открывать его из скрипта.

### Одиночная камера LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Относительные пути указывают на каталог, в котором запускается ваш скрипт (изменено в версии 1.3.0).** При использовании локального бэкэнда относительный путь `output_dir` (включая путь по умолчанию `"output"`) — а также другие пути, которые открывает бэкэнд (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — преобразуются относительно рабочего каталога вашего скрипта, поэтому `cam.capture("output/")` записывает данные в `<current folder>/output/`. В версии 1.2.0 установленный бэкенд преобразовывал их в свою собственную папку установки и выдавал ошибку «Доступ запрещён». При работе с удалённым бэкендом пути отправляются в том виде, в каком они введены.
{% endhint %}

### Синхронизированный массив — `connect_array`

`connect_array` — рекомендуемая точка входа для многокамерных установок. В нём запускается тот же поток интеллектуальной подготовки, что и в графическом интерфейсе: анализ сети, автоматический выбор уровня синхронизации, синхронизация времени по PTP, выбор формата пикселей для каждой камеры, инициализация автоэкспозиции и постановка на триггер GPIO. **Первый последовательный порт является ведущим** (он генерирует импульс аппаратного триггера); остальные — ведомые.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Добавьте `smart=True` к любому массиву захвата, чтобы дождаться стабилизации автоэкспозиции на всех камерах перед запуском триггера.

Нововведение версии 1.3.0: `ArraySession` также предоставляет синхронизированные кадры напрямую, без записи в файлы:

* `arr.grab()` возвращает по одному кадру с каждой камеры, полученным в результате одного триггера, в виде массивов numpy (`{serial: array}`); `.group_id` — это общий триггер, а `.errors` указывает любую камеру, которая не была включена.
* `arr.frame_stream(fps=5.0)` возвращает один такой набор за каждый такт для обработки в реальном времени в вашем собственном конвейере.
* `arr.continuous("output/")` запускает собственную непрерывную съёмку бэкэнда (режим «Непрерывный» в графическом интерфейсе) и возвращает `ContinuousHandle` с `status()`, `wait()` и `stop()` — используйте его вместо цикла захвата, когда необходимо сохранять каждый триггер с высокой частотой.

Информацию о режимах захвата (Одиночный / Непрерывный / Интервальный / Максимально быстрый), регистраторах, режиме «burst-to-video» и выравнивании массивов см. в [Справочнике по SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Датчик освещенности DAQ — `connect_daq_sensor`

Без аргументов `connect_daq_sensor()` автоматически определяет транспортный протокол (по приоритету: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Каждый фрейм содержит 135 точек `spectrum` (Вт/м²/нм после калибровки), флаг `is_saturated` и CIE `x`, `y`, `z`. Чтобы зафиксировать конкретный датчик или канал передачи, передайте один явный указатель:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Обратите внимание, что профили коррекции капа (`cap_id`) **не** являются параметром SDK — выбирайте их вместо этого с помощью `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Сохраненные проекты — `open_project`

Сохраненный проект Chloros сохраняет подключенное к нему оборудование (`cameras.json` + `sensors.json` наряду с `project.json`), а `chloros_sdk.open_project(path)` может одновременно переподключить свои камеры, массивы и датчики освещенности и запускать съемку по имени устройства. Датчики освещенности подключаются заново через работающий бэкенд Chloros с сохраненными настройками и пределом. См. раздел [Автоматизация проектов](reference/sdk-reference.md#project-automation--chlorosproject) в справочнике.

## Что дает установка только с помощью pip

Перед использованием аппаратных поверхностей проверьте флаги доступности на уровне модулей:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

На хосте, где установлен **только** `pip install chloros-sdk` и отсутствует пакет рабочего стола Chloros:

* `ChlorosLocal`, `process_folder` и `process_lattice_capture` **не** работают — им требуется бинарный файл бэкэнда, поставляемый в составе установщика рабочего стола.
* Помощники smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) являются чистыми HTTP-клиентами, поэтому они работают с бэкендом на другом компьютере — однако поставляемые бэкенды привязаны только к петлевому интерфейсу, поэтому вам необходимо самостоятельно перенаправить порт (например, `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) и передать `backend_url="http://127.0.0.1:5000"` вместе с `auto_start_backend=False`. См. [Режим удалённого бэкэнда](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Классы LATTICE с прямым доступом к аппаратному обеспечению (`LatticeCamera`, `CameraPool`, …) импортируются, но требуют наличия среды выполнения Arena SDK из пакета для настольных компьютеров — без неё `CAMERA_AVAILABLE` будет работать как `False`.
* Датчики DAQ управляются только через (туннелированный) бэкенд — `connect_daq_sensor()` или `proj.sensors` проекта; бэкенд открывает датчик на своей собственной машине.

## ИИ-помощники — `chloros-mcp`

SDK включает `chloros-mcp` — сервер MCP, который позволяет Claude Desktop, Claude Code, Cursor и другим клиентам MCP управлять Chloros: обрабатывать изображения и запускать камеры LATTICE, массивы и световые датчики DAQ. Его инструменты охватывают все возможности данного SDK. Он устанавливается вместе с SDK, поэтому для Claude Code настройка сводится к одной строке:

```bash
claude mcp add chloros -- chloros-mcp
```

Другие клиенты: [Использование Chloros с AI-помощниками](ai-assistants.md). Параметры и все инструменты: [Сервер MCP (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Требования к лицензии

Для доступа к SDK требуется активная учетная запись Chloros+ на любом платном тарифе — **Copper или выше**(Copper / Bronze / Silver / Gold); бесплатный тарифный план «Iron» не предоставляет доступа к SDK/CLI. Проверка осуществляется**на стороне сервера**: каждый запрос SDK должен содержать данные как о действующей сессии, так и о платном тарифном плане, в противном случае бэкенд возвращает `403` / `PLAN_UPGRADE_REQUIRED` (генерируемые как `ChlorosLicenseError` с помощью `ChlorosLocal` и как `ChlorosConnectError` с помощью `connect_*`). Вызывающий, выйдя из системы, получает `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — повторный запуск `chloros-cli login` устраняет первый случай, но не второй.

Использование в автономном режиме работает в течение льготного периода тарифного плана: уровень доступа считывается из кэша проверки сервером (5 минут) или из кэша подписанных, привязанных к устройству лицензий (30 дней для месячных тарифных планов; до истечения срока подписки для годовых). По истечении льготного периода тарифный план переходит в бесплатный режим, и доступ к SDK приостанавливается до тех пор, пока устройство не установит соединение с сервером хотя бы один раз. Код ошибки `chloros-cli status` остается доступным на бесплатном тарифном уровне, поэтому причина всегда видна. См. [Вход в Chloros+](chloros+-login.md).

## Исключения

Обработайте базовый класс для обработки «любых сбоев в работе Chloros»:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

Все исключения конвейера (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) являются производными от `ChlorosError`. Один нюанс: `ChlorosConnectError` — генерируется только `connect_camera` / `connect_array` / `connect_daq_sensor` — происходит от простого `Exception`, **а не** от `ChlorosError`, поэтому `except ChlorosError` его не перехватит. Полная иерархия приведена в [Справочнике по SDK](reference/sdk-reference.md#exceptions).

## См. также

* [Справочник по SDK](reference/sdk-reference.md) — полный набор API, оптимизированный для AI-помощников.
* [Справочник CLI](reference/cli-reference.md) — каждая подкоманда CLI соответствует вызову SDK.
* [Использование Chloros с AI-помощниками](ai-assistants.md) — рецепты настройки и подсказок `chloros-mcp`.
* [Скачать](download.md) — установщики для Windows и Linux.
