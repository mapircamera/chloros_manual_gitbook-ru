# Chloros Python SDK Справочная информация

**Версия:**

1.2.0**Создано:**

29.07.2026 19:19 ·**Обновлено:**

30.08.2026**Пакет:** `chloros-sdk` (PyPI)**Аудитория:** Оптимизировано для использования в LLM; удобно для чтения человеком.**Область применения:** Все общедоступные классы, функции и вспомогательные методы, предоставляемые `import chloros_sdk`, с примерами, которые можно скопировать и вставить, охватывающими обработку изображений, управления одной камерой, синхронизированных массивов, датчиков DAQ и автоматизации проектов.

Если вам нужны только основные моменты, перейдите к:
- [Установка и быстрое начало работы](#installation)
- [Smart-Connect для массивов LATTICE](#smart-connect-for-lattice-cameras)
- [Сеансы датчиков DAQ](#daq-sensor-sessions)
- [Автоматизация проектов](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Архитектура за 60 секунд

SDK представляет собой тонкий слой Python, наложенный на бэкенд Chloros (тот же сервер Flask, который используют настольный графический интерфейс и CLI). Для автоматизации необходимо импортировать `chloros_sdk` и вызывать методы высокого уровня; «под капотом» каждый вызов преобразуется в запрос HTTP к локальному бэкенду на порту 5000 — `http://127.0.0.1:5000/api/...` (намеренно не `localhost`, который сначала разрешается в `::1` на Windows и занимает ~2 с на каждый запрос при работе с бэкендом, поддерживающим только IPv4). Бэкенд владеет аппаратным пулом — камерами, датчиками DAQ, профилями выравнивания, буферами кадров — поэтому скрипты SDK могут сосуществовать с графическим интерфейсом, не конкурируя за последовательные порты или пропускную способность сетевых карт.

Вы будете использовать три рабочих поверхности:

1. **`ChlorosLocal` + свободные функции** (`process_folder`, `process_lattice_capture`) — конвейер обработки изображений. Обработайте всю папку с калибровкой, дебайеризацией и экспортом индекса одним вызовом Python.
2. **Модули Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — открытие постоянного сеанса бэкэнда для рабочего оборудования. В качестве графического интерфейса используется тот же поток «smart-prep», что и в графическом интерфейсе: проверка сети, автоматический выбор уровня, PTP, инициализация AE, настройка триггера GPIO.
3. **`ChlorosProject` / `open_project`** — Загрузить сохраненный проект (папку с файлами `cameras.json` + `sensors.json` + `project.json`), подключить всё сразу и выполнять захват данных с помощью именованных дескрипторов.

Поверхности 1 и 2 **автоматически запускают локальный бэкенд**, если он ещё не находится в режиме прослушивания (тот же бинарный файл из комплекта, который запускают GUI/ CLI) — поэтому простой скрипт работает из новой оболочки без необходимости сначала запускать бэкенд. Передайте `auto_start_backend=False`, чтобы отключить эту функцию (например, при указании на удалённый бэкенд, который никогда не запускается). См. [Автозапуск бэкенда](#backend-auto-start). Surface 3 ведёт себя иначе: `open_project()` не принимает параметр `auto_start_backend`, а `connect_all()` никогда не запускает бэкенд — он один раз проверяет `http://127.0.0.1:5000` и, если никто не отвечает, без предупреждения переходит к прямому (без бэкенда) управлению устройством `lattice_sdk`. Только `proj.process()` и `stream(..., overlays=True)` лениво создают `ChlorosLocal()` (который запускается автоматически).

Все три имеют ограничение по авторизации: запустите `chloros-cli login` один раз на машине или войдите в систему через графический интерфейс рабочего стола. Вызовы SDK без действительной сессии вызывают ошибку `ChlorosAuthenticationError`.

Требования:
- Python версии 3.7+ (согласно описанию пакета; разработано и протестировано на версии 3.10)
- Локально установленная программа «Chloros Desktop» (бинарный файл бэкэнда входит в состав установщика)
- Активный логин Chloros+. Минимальный уровень доступа к SDK / CLI — **Copper**или выше (Copper / Bronze / Silver / Gold); бесплатный уровень**Iron**не предоставляет доступа к SDK / CLI. Это ограничение применяется**на стороне сервера-уровне**: каждый запрос с флагами SDK / CLI должен содержать как активную сессию, так и платный тарифный план, иначе бэкенд возвращает `403` с `error_code: PLAN_UPGRADE_REQUIRED` (отображается как `ChlorosLicenseError` с помощью `ChlorosLocal` и как `ChlorosConnectError` с помощью `connect_*`). Вызов от пользователя, вышедшего из системы, возвращает `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) вместо этого — эти два кода различаются, поскольку повторный запуск `chloros-cli login` устраняет ошибку первого кода, но не может устранить ошибку второго.
- Офлайн-использование поддерживается в течение льготного периода тарифного плана: уровень доступа считывается из кэша серверной проверки (5 минут) или из кэша подписанных, привязанных к устройству лицензий (30 дней для месячных тарифных планов и до истечения срока подписки для годовых). По истечении этого льготного периода тарифный план переходит в бесплатный режим, а доступ к SDK / CLI приостанавливается до тех пор, пока компьютер не сможет хотя бы один раз подключиться к серверу. `chloros-cli status` (`GET /api/license-status`) остаётся доступным на бесплатном тарифе, поэтому причина видна — это единственный маршрут SDK / CLI, не подпадающий под ограничения тарифного плана.
- Windows 10/11 64-битная версия, **Ubuntu 22.04 LTS или более поздние версии**, либо Jetson (JetPack 6). Ubuntu 20.04**не** поддерживается: зависимости `.deb` производны от того, с чем связывается бэкенд, включая `libc6 (>= 2.34)`, а в версии Focal поставляется glibc 2.31.

---

## Установка

Python SDK представляет собой тонкий слой Python, наложенный на бэкенд Chloros. Для всего, что выходит за рамки нескольких рабочих процессов, связанных исключительно со сбором данных (DAQ), необходимо **локально установить пакет Chloros для настольных систем** (установщик Windows или Linux `.deb`) — именно он предоставляет бинарный файл бэкэнда, среду выполнения Arena SDK для камер LATTICE и наборы для калибровки.

Последние версии для скачивания: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Шаг 1 — Установите пакет платформы Chloros

#### Windows (.exe)

1. Загрузите файл `Chloros-Setup-x.y.z.exe` со страницы загрузки.
2. Запустите установщик и следуйте инструкциям мастера. Путь установки по умолчанию — `C:\Program Files\MAPIR\Chloros\`.
3. Запустите Chloros хотя бы один раз и войдите в систему, используя свою учётную запись Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Шаг 2 — Установите Python SDK

**Установщик Chloros поставляется с соответствующим пакетом SDK wheel.** Каждый установщик Windows и пакет .deb с Linux размещает на диске файл `chloros_sdk-X.Y.Z-py3-none-any.whl`, который точно соответствует версии графического интерфейса / CLI / бэкэнда. Вам не нужно следить за обновлениями на PyPI, чтобы оставаться в синхронизации.

#### Windows

Установщик автоматически запускает файл `pip install` с использованием входящего в комплект файла wheel и системного файла Python (предпочтительно используется запускатель `py.exe`, в качестве запасного варианта — `python -m pip`). Никаких действий не требуется — `import chloros_sdk` работает в вашей среде Python после успешной установки. Если на компьютере отсутствует файл Python, установщик незаметно пропускает этот шаг, а графический интерфейс и файл CLI продолжают работать.

#### Linux (.deb)

Пакет .deb размещает файл wheel в каталоге `/usr/lib/chloros/sdk/`. Файл `postinst` выводит точную команду — дистрибутивы, соответствующие PEP 668, по умолчанию запрещают глобальную запись pip, поэтому мы не выполняем автоматическуюустанавливаем:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Для развёрток Jetson в изолированной среде (air-gapped) этот процесс проходит полностью в автономном режиме — файл wheel уже находится на диске.

#### Общедоступный PyPI

Для хостов, использующих только pip (без установленного настольного пакета Chloros; рабочие процессы с удалённым бэкэндом или только с DAQ):

```bash
pip install chloros-sdk
```

PyPI обновляется при сборке установщиков для релизных версий, поэтому опубликованный файл wheel соответствует последнему стабильному выпуску. Разработческие сборки (например, `1.1.4.dev1`) поставляются только через входящий в комплект установщика файл wheel.

#### Проверка

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **Требуется подписка на Chloros+.** Для всех вызовов SDK требуется активный логин Chloros+. Запустите `chloros-cli login user@example.com 'YourPassword'` один раз на каждом компьютере; учетные данные кэшируются в `~/.chloros/`.

### Нужен ли мне пакет для настольных компьютеров?

Одного только пакета pip **не** достаточно для большинства рабочих процессов. Вот что требуется для каждой рабочей поверхности SDK:

| Рабочая поверхность SDK | Требуется ли пакет Desktop? | Почему |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Да** | Автоматически-запускает бинарный файл бэкэнда в `/usr/lib/chloros/chloros-backend` (Linux) или `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Да**(локальный)**/ Нет**(удалённый) | Чистые клиенты HTTP через бэкенд. Локальный бэкенд → требуется пакет для рабочего стола. Удалённый бэкенд → `backend_url=`**через туннель** (см. Удалённыйрежим бэкэнда — поставляемые бэкэнды привязываются только к петлевому интерфейсу). |
| `ChlorosProject` / `open_project` | **Да** | Запускает сохраненные проекты через бэкэнд. |
| Прямые классы LATTICE (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Да** | Требуется нативная среда выполнения Arena SDK, входящая в состав пакета для настольных компьютеров. В противном случае при импорте `CAMERA_AVAILABLE` соответствует `False`. |
| Классы Direct DAQ (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Нет** | Чистый Python поверх pyserial/bleak/zeroconf. Среда, использующая только pip, может управлять устройствами сбора данных (DAQ) от начала до конца. |

### Режим удалённого бэкэнда (хост, использующий только pip, через туннель)

> **Поставляемый бэкенд недоступен по локальной сети.** Производственные
> сборки привязываются только к лупбэку (оба семейства лупбэков) и категорически отказывают в
> единственном режиме, не связанном с лупбэком (`CHLOROS_CLOUD_MODE`), поэтому
> `backend_url="http://<lan-ip>:5000"` **не может работать с установленным
> Chloros** — этот шаблон работал только с бэкендом source/dev.
> Чтобы управлять бэкендом на другой машине, самостоятельно перенаправьте его
> порт loopback и укажите SDK на туннель:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Хосты без монитора / CI / робототехники могут использовать одну машину с полной установкой рабочего стола в качестве «Chloros-сервера», а везде остальные — `pip install chloros-sdk`, но транспорт между ними — это организованный пользователемустроенный выше туннель, а не прямой LAN-URL.

> **Известное ограничение — `ChlorosLocal` не поддерживает работу исключительно через pip.** В настоящее время `ChlorosLocal(backend_url=BACKEND)` определяет локальный бинарный файл бэкэнда в своём конструкторе *до* проверки URL и выдает ошибку `ChlorosBackendError` («Chloros: бэкэнд не найден…») , если не установлен пакет для рабочего стола — даже при наличии доступного удалённого бэкенда. Только интерфейс smart-connect, указанный выше (`connect_camera` / `connect_array` / `connect_daq_sensor`, а также `analyze_array_network` и вспомогательные программы `list_*` / `discover_*`) работают с хоста, на котором установлен только pip.

### Рабочий процесс, включающий только сбор данных (хост, на котором установлен только pip)

Если вам нужны только датчики сбора данных (DAQ) и вы не используете камеры LATTICE или обработку изображений, пакет pip является самодостаточным:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Для работы с DAQ напрямую с аппаратным обеспечением не требуется ни бэкэнд, ни .deb, ни вход через Chloros+.

---

## Быстрый старт

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Индекс верхнего уровня API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Обработка изображений — `ChlorosLocal`

Основной класс конвейера. Запускает бэкенд при первом использовании, создаёт и настраивает проекты, отслеживает ход выполнения, возвращает сводные отчёты по итогам выполнения.

### Конструктор

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

### Методы

| Метод | Описание |
| --- | --- |
| `create_project(project_name, camera=None)` | Создание нового проекта (по желанию с использованием шаблона камеры, например `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Импорт изображений в форматах RAW/TIF/JPG/DNG **и записи датчика освещённости `.daq`**. Возвращает `count` (изображения) и `scan_count` (записи). Выдает предупреждение, только если в папке нет ни того, ни другого. |
| `export_light_sensor(daq=True, csv=True)` | Запись откалиброванных файлов `.daq` + `.csv` для каждой записи датчика освещенности в проекте в файл `<project>/Light Sensor/`. См. [Записи датчика освещенности](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Установить параметры обработки. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Запустить конвейер обработки. Возвращает `{"status": "complete", "async": False}`, а также ключ `summary`, если бэкенд его предоставляет — см. [Сводка и подсказки после запуска](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Проверить состояние бэкэнда. |
| `logout()` | Очистить кэшированные учетные данные. |
| `shutdown_backend()` | Прекратить работу бэкенда (если запущен с параметром `SDK-started`). |
| `discover_cameras()` | Обнаружить камеры LATTICE **через бэкенд данного экземпляра** (`/api/camera/discover`). Возвращает список словарей (`serial`, `model`, `ip`, …) — той же структуры, что и в графическом интерфейсе пользователя (GUI)/ CLI. Пустой список, если ничего не найдено или бэкенд недоступен. |
| `camera_capture(output_dir, format="tiff", **settings)` | Захват одного кадра**через бэкенд**(автозапуск по данному дескриптору), чтобы он получил ту же подготовку, что и в графическом интерфейсе/ CLI (по умолчанию 12 бит, повторное использование пула, встроенные калибровочные метаданные). Определите цель с помощью `serial=` или `device_index=`; передайте `exposure`/`gain`/`pixel_format`/`preset` в виде `**settings`. Возвращает словарь устаревших метаданных (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Выдаёт кадры предварительного просмотра с наложенным оверлеем от объединенной камеры — лёгкий MJPEG-клиент через маршрут `/api/camera/<serial>/stream-annotated` бэкэнда (зебра / сетка / перекрестье / гистограмма / пикинг / точечное рисование на стороне сервера). `decode=True` возвращает массивы BGR; `False` возвращает необработанные байты JPEG. Также доступно на уровне проекта как `ChlorosProject.stream(overlays=True)`. |

Используйте в качестве контекстного менеджера для гарантированной очистки:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Записи с датчиков освещенности — откалиброванные `.daq` + `.csv`

Данные с DAQ-U / DAQ-M / DAQ-E можно записывать **без** пакета калибровки. Именно
это по умолчанию делают общедоступные [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
регистраторы (`record_daq.py`) делают по умолчанию: они записывают необработанные данные датчика и помечают
файл так, чтобы Chloros извлекал заводскую калибровку этого датчиказаводскую калибровку **по серийному номеру** — сначала из локального кэша
, затем из облака MAPIR — и применяет её при импорте.

Chloros записывает результат обратно в виде двух продуктов на каждую запись под
`<project>/Light Sensor/`:

| Продукт | Что это |
| --- | --- |
| `<name>_calibrated.daq` | Архив, поддающийся повторной обработке — та же схема, что и у записи в реальном времени, теперь с указанием пакета, который его сгенерировал. Повторный импорт **не** приводит к повторной калибровке. |
| `<name>_calibrated.csv` | Спектральная интенсивность излучения в Вт/м²/нм по собственной сетке длин волн датчика, по одной строке на каждое измерение, плюс фотометрические столбцы (общая мощность, фотопические/скотопические люксы, PPFD и его разбивка по синему/зелёному/красному спектрам, пиковая длина волны). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Только для датчиков без набора данных (DAQ-A).** Исходные спектральные отсчёты датчика — *не* интенсивность излучения. См. ниже. |

`process()` выполняет этот экспорт в качестве одного из этапов. Для этого **не** требуются изображения:
самостоятельное использование датчика освещённости представляет собой полноценный рабочий процесс, и такой проект по определению не содержит
изображений.

**Записи DAQ-A экспортируются в виде необработанных значений.** Семейство DAQ-A появилось до внедрения системы
связок по серийным номерам и не имеет связки для извлечения — вместо этого оно калибруется в полевых условиях по
мишени отражательной способности, поэтому в ней никогда не было необходимости. Эти записи экспортируются
с корнем `_raw`, а не `_calibrated`: другое имя файла, а не флаг
внутри файла, поскольку информация должна сохраниться при отправке по электронной почте в виде простого имени. В
заголовке `.csv` указано `raw spectral sensor counts (NOT irradiance)` и содержится предупреждение о том, что
значения сопоставимы **в пределах** файла — именно для этого и используется калибровка по мишени
— а не между датчиками. Фотометрические столбцы, зависящие от мощности (общая мощность,
фотопический/скотопический люкс, PPFD), возвращаются как **NULL**, а не интегрируются из отсчётов.

DAQ-U / DAQ-M / DAQ-E, пакет для которых просто не удалось загрузить, по-прежнему **пропускаются**,
а не записываются в необработанном виде: в этом случае пакет существует, и «повторно подключиться и переобработать» — реальный совет.

Записи устаревшей версии **v1.01 / v1.02** (их записывает DAQ-A-SD) не содержат эпохи для каждого отдельного считывания,
только время записи файла. Модуль сопоставления изображений ↔ нисходящего излучения по-прежнему отклоняет их — сопоставление
кадра со временем записи привело бы к незаметной ошибке — но экспортер их считывает, и
CSV выводит `clock=daq_created_on`, чтобы продукт указывал, по какому часовому сигналу он работает.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Запись, пакет калибровки которой невозможно получить (в автономном режиме или если для датчика
в файле отсутствует калибровка), отображается как `skipped` **с указанием причины**. Она никогда не
записывается как «откалиброванный» файл, содержащий необработанные данные подсчёта — подключитесь к Интернету и
запустите процесс заново, после чего экспорт завершится.

### Обратные вызовы о ходе выполнения

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Сводка и подсказки после завершения

По завершении `process()` извлекает `GET /api/processing-summary` и присоединяет его тело в виде `result["summary"]`. Извлечение выполняется по принципу «по мере возможности» и никогда не блокирует успешный возврат — если сводка недоступна, `process()` переходит к простой форме `{"status": "complete", "async": False}`. Каждая запись в `summary["hints"]` — полные предложения с предложенными мерами по исправлению, например, почему запуск дал нулевой результат — также повторно выводится в виде Python `UserWarning`, поэтому запуски с нулевым выводом являются самодиагностирующимися, даже если вы никогда не просматриваете словарь:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` представляет собой машиночитаемую часть:

| Ключ | Что учитывается |
| --- | --- |
| `models` | Группы камер в серии. |
| `images_in_groups` | Исходные изображения по этим группам. |
| `targets_found` | Обнаруженные мишени отражения. |
| `images_calibrated` | Изображения, на которых была выполнена калибровка запуска. |
| `exported_files` | **Файлы выходных изображений, созданные в ходе запуска.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Записи датчиков освещённости, специально учтённые отдельно — они поступают с другого этапа и существуют даже для сеансов, в которых изображений нет вовсе, поэтому их включение привело бы к тому, что запуск, в котором собирались только данные, выглядел бы так, будто из него были экспортированы изображения. |

Наряду с ними: `summary["output_dirs"]` (каждый каталог, в который были записаны данные),
`summary["light_sensor_export"]`, `summary["stopped"]` (имеют значение «true», если пользователь прервал
запуск, поэтому частичные подсчёты несчитаться завершённым запуском с недополученным объёмом данных), и
`summary["groups"]` (разбивка по группам).

`exported_files` записывается конвейером **по мере записи**, а не сканируется из
объектов образов проекта впоследствии. Параллельные и GPU-стратегии создают свои собственные объекты образов
(в рабочих подпроцессах для GPU-путей), поэтому старое сканирование сообщало
`0 file(s) written` для каждого такого запуска, а затем выдавало подсказку о нулевом количестве экспортов — при запусках,
в которых всё работало нормально. Если вы создаете скрипт с учетом этого номера, то теперь исправный параллельный запуск
сообщает ненулевое значение.

Сообщения о пропусках датчика Light-sensor указывают причину, которую считыватель фактически установил для каждого файла —
нечитаемая схема, отсутствующий пакет, ошибка записи — **без дублирования**, поэтому двадцать файлов,
пропущенных по одной причине, отображаются как одна причина, а не как двадцать повторений этой причины.

> **`process()` не генерируется, если запуск не производит изображений.** Это единственное место, где SDK и
> CLI намеренно различаются: `chloros-cli process` трактует «были запрошены продукты, ни один из них не был
> записан» как сбой и завершается с ненулевым кодом, тогда как SDK возвращается в нормальном режиме и сообщает об
> этом состоянии через `summary` / hints. Если ваш конвейер должен останавливаться при пустом прогоне, проверьте это
> самостоятельно — просмотрите `summary` (или посчитайте файлы в папке проекта), а не полагайтесь на
> отсутствие исключения. Обычными причинами являются папка входных данных, которая не была распознана как
> захват, и продукты, пропущенные как неприменимые для имеющихся камер (например, яркость от камер, поддерживающих только режим «RGB»
>).

### Удобные функции

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Поддерживаемые значения

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Радиометрический выход (мультиспектральный конвейер LATTICE)

Уровень экспорта мультиспектральных данных (M3C/M3M) конвейера `process` — `reflectance` (по умолчанию), `radiance`, `sensor-response` или `all` (каждый применимый режим для каждого изображения) — сопоставляется с настройкой **«Радиометрический выход»** . Для `configure()` предусмотрено специальное ключевое слово:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

Расширенный «запасной выход» — запись ключа проекта `"Radiometric output"` через `custom_settings` — по-прежнему работает, но помните, что он заменяет весь блок настроек (см. предупреждение ниже):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (по умолчанию) делит яркость камеры на **нисходящий поток данных DAQ, сопоставленный по временной метке**, который автоматически определяется из записанного `.daq` (DAQ-U/M/E)**или из собственного `.csv` DAQ-M**, найденного вместе с изображениями; любой пакет калибровки для отдельной камеры или DAQ, отсутствующий локально,**автоматически загружается из AWS** при первом использовании. CLI предоставляет доступ к этим параметрам в виде переключателей для продуктов по типу в файле `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **заменяет** весь блок вычисляемых настроек (по замыслу он игнорирует остальные ключевые слова и проверку `configure()`). При его использовании включите все ключи `Project Settings`, которые вам нужны, как показано в примере выше.

---

## Smart-Connect для камер LATTICE

Постоянные сеансы бэкэнда для аппаратного обеспечения в режиме реального времени. Используются те же конечные точки, что и в графическом интерфейсе, поэтому поведение одинаково на SDK / CLI / в графическом интерфейсе.

### Одна камера — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Сигнатура `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### Методы `CameraSession`

| Метод | Описание |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Чтение узлов GenICam; возвращает `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Запись узлов по дружественному имени (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Захват **одного** кадра. Возвращает список из одного элемента, содержащий словари метаданных кадра. (Серийная/многокадровая съемка была удалена — для съемки серии вызовите `capture()` в цикле.) |
| `disconnect()` | Освобождение из пула. Не выполняет никаких действий, если мы подключились к уже открытому сеансу. |

Управление экспортом `capture()` (та же модель, что и для массива + графический интерфейс):

- `processing` / `levels` — `processing="all"` сохраняет каждый соответствующий тип экспорта; `levels=["raw","radiance"]` сохраняет только те (переопределяет `processing`). Пропустите оба параметра для использования значений по умолчанию бэкэнда.
- `force_daq=True` — сохраняет назначенное значение с DAQ/DLS в виде файла-приложения `.daq` даже при захват только в необработанном виде, чтобы кадр можно было позже переобрабатывать в отражательную способность/индекс. Не выполняется, если DAQ не подключен.

### Синхронизированный массив — `ArraySession` (Smart-Prep)

`connect_array` является **рекомендуемой точкой входа** для многокамерных установок. Она запускает полный цикл «Smart-Prep» через графический интерфейс пользователя:

1. **Анализ сети** (`/api/camera/array/recommend`) — находит максимальный размер кадра, который помещается в уровень sim-emit без потери кадров.
2. **Автоматический выбор уровня** — `sim-capture-sim-emit`, если кабель это поддерживает; в противном случае — `sim-capture-ftd-stagger` или `slip-emit-and-capture`.
3. **Автоматическое уменьшение**— незаметно уменьшает размер кадра / увеличивает биннинг, когда канал не может поддерживать запрашиваемое разрешение.**Эта защитная мера не распространяется на совокупную перегрузку**: проблему слишком большого количества камер для канала связи невозможно решить за счёт уменьшения размера кадров — см. [Перегрузка](#over-subscription-the-per-cam-floor).
4. **PTP включен** по умолчанию — временные метки между камерами сопоставимы с точностью до микросекунд.
5. **Автоматический выбор формата пикселей для каждой камеры** — камеры RGB → `BayerRG8`, мультиспектральные → `BayerRG12`.
6. **Инициализация автоэкспозиции** — фиксирует текущее состояние автоэкспозиции каждой камеры, чтобы подключение не сбрасывало экспозицию в процессе работы.
7. **Настройка триггера GPIO** — `connect_array` активирует все камеры (`TriggerMode=On`, `TriggerSource=Line2`), чтобы импульс ведущей камеры управлял ведомыми через кабель M8. Этот шаг действует только для массива: отдельная камера, запущенная с помощью `LatticeCamera`, вместо этого работает в автономном режиме.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Сигнатура `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Значения `force_tier`:
- `"sim-capture-sim-emit"` — истинная синхронность (все камеры срабатывают на одном и том же фронте тактового сигнала).
- `"sim-capture-ftd-stagger"` — гибкое чередование во временной области (камеры генерируют сигналы с небольшим смещением по времени, так что пакеты сериализуются в линии передачи).
- `"slip-emit-and-capture"` — последовательный захват по каждому камерному модулю (без временной синхронизации; единственный вариант, когда ни один размер кадра не подходит для синхронного режима).

`wire_ceiling_mbps` переопределяет **устойчивый бюджет канала хоста** в МБ/с — единственное
число, от которого зависит распределение ресурсов всего массива. Оставьте значение `None`, чтобы использовать автоматически определённое
значение. Уменьшите его, если массив сообщает о GVSP-поврежденные кадры: автоматическое значение вычисляется
на основе заявленной скорости соединения сетевой карты, которая завышает показатели USB-адаптеров, узких линий PCIe и
загруженных общих магистралей — и эта завышенная оценка проявляется в виде повреждённых кадров, а не в виде
заметно медленного соединения. Это значение сохраняется в блоке захвата массива проекта, поэтому
повторное открытие или последующее изменение параметра `connect_array` восстанавливает его, как и любой другой параметр массива.
См. [Состояние массива](#array-health--which-subsystem-is-losing-frames).

#### Переподписка (минимальный порог на камеру)

Алгоритм Sim-emit pacing выделяет каждой камере долю бюджета канала, защищённого от коллизий, с минимальным порогом **8 МБ/с на камеру**(`per_cam_floor_bps`). Как только `N × floor` превышает предел, массив**переподписывает канал**— режимом отказа является потеря пакетов GVSP, а не снижение частоты кадров — и исправить это с помощью изменения размера кадра невозможно:**бинирование и ROI уменьшают количество байтов в кадре, а не количество байтов в секунду с регулируемой скоростью передачи**, которое сравнивает агрегатная проверка. Практические предельные значения для полного разрешения на хосте 1 Гбит/с:**6 камер при MTU 1500, 9 — с джамбо-кадрами** (`max_cams_collision_safe` в ответе анализа указывает предельное значение для вашего канала). Способы устранения: меньшее количество камер, использование джамбо-кадров по всей длине канала или более быстрая сетевая карта.

- Ответы `analyze_array_network()` и `/api/camera/array/connect` содержат `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` и `per_cam_floor_bps`. Если значение `oversubscribed` равно true, то при проекции **обнуляет поля fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`), а не сообщает вводящую в заблуждение информацию о низкой, но работоспособной скорости.
- `POST /api/camera/array/connect` принимает параметр тела `pin_resolution` (**только HTTP — не ключевой аргумент SDK**; `connect_array` не предоставляет его). Фиксация удаляет защитную сетку понижения биннинга, поэтому перегруженное соединение с установленным `pin_resolution`**категорически отклоняется** с ошибкой, в которой перечислены все возможные меры устранения. Без фиксации соединение продолжает понижение, но выдает предупреждение о том, что уменьшение не сможет очистить агрегат.
- Лазейка для тестирования: установите `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` в среде бэкэнда, чтобы понизить уровень отказа до громкого предупреждения — вы всё равно установите соединение в любом случае и примите потерю пакетов.

#### Состояние массива — какая подсистема теряет кадры

`GET /api/camera/array/<array_id>/capability` отслеживает активный блок `health` на
подключённом массиве, переоцениваемый в скользящем **10-секундном** окне. Он разбивает потери кадров
на две причины, требующие противоположных мер, вместо одного показателя «неполноты», который
не указывает ни на одну из них:

| Поле | Что это означает | Какая подсистема |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (по последовательному порту) | Фрейм **прибыл, но имел структурные повреждения**— потеря пакетов GVSP. |**Сеть**: пропускная способность кабеля, темп передачи, кольцо приёма сетевой карты, MTU |
| `never_arrived_rate_pct` (по серийному номеру) | Кадр **вообще не поступил**— камера не сработала или данные не были отправлены. |**Триггер / синхронизация**: кабель M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Наихудший показатель по каждой камере. | — |
| `per_cam_rate_pct` | Совокупный показатель неполных кадров на камеру (обе причины вместе). | — |
| `stable_for_seconds` | Как долго каждая камера оставалась на уровне ниже 0,01 %. | — |

Наряду с `health` в том же отчёте указано значение, на котором зависает весь выделенный ресурс:

| Поле | Что это означает |
| --- | --- |
| `wire_ceiling_mbps` | Действующий устойчивый бюджет пропускной способности хоста, МБ/с. |
| `wire_ceiling_source` | Источник этого значения, в текстовом формате — например, `USB-capped 200 MB/s (was theoretical 1062; …)` или `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true`, когда `wire_ceiling_mbps=` установил это значение. |
| `nic_is_usb` | `true` для USB-адаптера Ethernet. |

Для этого конечного пункта нет оболочки SDK — читайте его напрямую:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Чтение:** ненулевое значение `gvsp_corrupt_rate_pct` при значении `never_arrived_rate_pct`, равном 0, означает, что
триггер и синхронизация по кабелю идеальны, а 100 % потерь приходятся на сетевой путь — уменьшите
значение `wire_ceiling_mbps` и выполните повторное подключение. Обратная картина указывает на проблему в синхронизационном кабеле или
линии триггера.

> **`target_fps` не является показателем повреждённых фреймов.** Параметр GevSCPD устанавливается однократно при
> подключении, поэтому снижение частоты триггера изменяет рабочий цикл, а не
> скорость одновременной передачи пакетов. Измеренное 5-кратное снижение нагрузки не дало улучшения, тогда как
> снижение максимальной пропускной способности кабеля с 240 до 200 МБ/с позволило снизить долю повреждённых кадров на той же установке с 10,4 % до
> 0,00 %.

> **Автоматическое сокращение потока в середине передачи недоступно в прошивке TRI032S.** Работающий массив не может
> исправить это самостоятельно; отсоедините и подключитесь заново, чтобы механизм выбора при подключении перепланировал работу с учётом
> нового предельного значения.

**USB-адаптер Ethernet ограничивается пробником до 200 МБ/с** независимо от его
номинальных характеристик: таблица эффективности, преобразующая скорость канала в устойчивое значение,
основана на стандарте PCIe, а USB-сетевая карта объявляет свою скорость Ethernet-канала, будучи ограниченной
USB-шиной и её драйвером. Ограничение является абсолютным, а не относительным — USB-адаптер 1 Гбит/с
обеспечивает скорость ~80 МБ/с и не подпадает под это ограничение.

#### Методы `ArraySession`

| Метод | Описание |
| --- | --- |
| `status(timeout=10.0)` | Live `{fps, ptp, frame_count, last_error, …}`. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Одна синхронизированная группа захвата. Возвращает `CaptureResult` (список словарей кадров + `.skipped`). Экспортные параметры приведены ниже. |
| `capture(..., smart=True)` | **Интеллектуальная съемка** — ожидает стабилизации AE на всех камерах, затем запускает съемку. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Самая быстрая съемка: только необработанные данные + назначенное значение считывания DAQ (+ свободный комбинированный индекс). Отражает кнопку «Самая быстрая съемка» в графическом интерфейсе. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Одиночный / Непрерывный / Интервальный режим в одном ограниченном цикле. Возвращает `list[CaptureResult]`.**Требует `count` и/или `duration_s`** для завершения работы (в режиме «SDK» нет Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Запуск записи живого изображения с комбинированным индексом в формат видео/GIF → `RecorderHandle`. Один комбинированный рекордер на массив. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Запуск серии снимков в формате raw-Bayer с высокой частотой кадров → `RecorderHandle`. Переработка в автономном режиме с помощью `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Офлайн-переработка сохраненной серии снимков в формате RAW в откалиброванное видео. Блокирует выполнение до завершения (`wait=True`) и возвращает `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Опрос задания автономной сборки: `{running, result, error, burst_dir}`. |
| `disconnect()` | Освободить весь массив. |

`capture()` — управление экспортом (та же конечная точка, что и в графическом интерфейсе пользователя/CLI):

- `processing` / `levels` — `processing="all"` (или `levels=["raw","radiance",…]`) сохраняет все применимые типы экспорта для каждой камеры; одно значение `processing` сохраняет только этот уровень.
- `aligned=True` — деформировать не-сырой экспорт каждого элемента в соответствии с [профилем выравнивания](#array-alignment) массива (с совместной регистрацией); необработанные данные остаются без деформации, но несут в себе трансформацию в метаданных. Если массив не имеет профиля, используется режим без выравнивания (с отображением предупреждения в поле `alignment` результата).
- `render_index=False` — пропустить наложение индекса растительности для каждой камеры; по умолчанию оно отображается там, где настроено.
- `force_daq=True` — сохранять назначенное значение DAQ/DLS в виде файла-сайдкара `.daq`, даже если ни один из выбранных уровней в этом не нуждается.

**Сжатие TIFF (регулятор HTTP -only):**`ArraySession.capture()` не отправляет ключ `compression`, поэтому применяется значение по умолчанию бэкенда — `POST /api/camera/array/capture` считывает параметр тела `compression`, по умолчанию используется `"deflate"` (zlib L1 без потерь + горизонтальный предиктор, ~4,1 МБ на кадр в полном разрешении). `"none"` записывает несжатые данные (~6,3 МБ/кадр) с**~5-кратной скоростью записи** — оба формата являются без потерь и считываются одинаково при импорте. В классе `SDK` для этого не предусмотрено аргументов; обходным путём является использование ``chloros-cli lattice array-capture --compression none`` или необработанного `HTTP`. DEFLATE также удерживает GIL `Python`, поэтому сжатая запись не параллелизуется между потоками записи для каждой камеры — для непрерывной съемки в полном разрешении с 8 камерами со скоростью датчика требуется `compression: "none"`. Подробности: [CLI Справочник → array-capture](cli-reference.md).**Переопределения экспорта для отдельных элементов (только HTTP):**тот же конечный пункт также принимает `exclude_serials` (список — удаление элементов из сохраненного набора; массив по-прежнему запускается как одна синхронизированная группа, а исключенные элементы возвращаются в `excluded`), `serial_levels` (переопределения на уровне камеры `{serial: [level tokens]}`), а также `serial_index` (переопределения наложения индексов на камеру `{serial: bool}`). Это параметры тела GUI-параметры тела, соответствующие параметрам графического интерфейса, и**пока не являются аргументами kwargs SDK**; для элементов, отсутствующих в таблицах, используются значения, распространяющиеся на весь массив: `levels` / `render_index`.

##### Проверка пропущенных камер — `CaptureResult.skipped`

`ArraySession.capture()` возвращает `CaptureResult`, который является подклассом `list`: можно перебирать его, индексировать, применять к нему `len()` — все существующие шаблоны продолжают работать. Новый код может проверять атрибут `.skipped`, чтобы узнать, какие камеры были исключены и почему. Наиболее распространённый случай — это камеры RGB в массиве со смешанными фильтрами, когда запрашивается `processing="radiance"` или `"reflectance"` — радианс на пиксель Байера не имеет смысла для широкополосного датчика, поэтому бэкенд пропускает эти камеры, вместо того чтобы генерировать бессмысленные данные.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Токены причин следуют шаблону `<level>-not-applicable-to-rgb-cam` (по одной записи на каждый пропущенный уровень, каждая из которых содержит `level`). Пропуски, связанные с коэффициентом отражения: `reflectance-skipped-no-fresh-dls` (отсутствуют свежие данные о нисходящем излучении), `reflectance-skipped-bound-daq-unavailable (…)` (не удалось установить связь с подключенным устройством сбора данных) и `dls-uncalibrated-band-<nm>` — диапазон в основном лежит за пределами радиометрически откалиброванного диапазона светового датчика устройства сбора данных (~374–974 нм), поэтому абсолютное деление по отражательной способности на основе DAQ отклоняется, и кадр явно переходит на режим работы по характеристикам датчика. Среди поставляемых моделей только F988 вызывает эту ошибку; поддерживаемый режим работы этой камеры — рабочий процесс с панелью отражательной способности.

Уровни `processing`:

| Уровень | Выход |
| --- | --- |
| `"raw"` | Одноканальный Байер (монохромные камеры: один диапазон) напрямую с датчика. |
| `"debayered"` *(по умолчанию SDK)* | 3-канальный BGR с помощью билинейной демозаики (монохромные камеры: 1-канальный оттенок серого). |
| `"radiance"` | float32 Вт/м²/ср/нм через полную радиометрическую цепочку. Только мультиспектральный режим — камеры RGB пропускаются. |
| `"reflectance"` | uint16 0,.32768 (готово к работе с Pix4D); требует сопряжения с DAQ в режиме реального времени для абсолютной калибровки. Только для мультиспектральных данных. |
| `"display"` | Полная цепочка, соответствующая предварительному просмотру в графическом интерфейсе (CCM + WB + гамма в соответствии с профилем камеры). |
| `"all"` | **Один файл на каждый соответствующий уровень** для каждой камеры (соответствует настройке GUI «Захватить всё» / CLI по умолчанию). Возвращаемый файл `CaptureResult` содержит по одному словарю кадра на файл `(cam, level)`, с указанием уровня в каждом словаре; неприменимые уровни отображаются в файле `.skipped`. Показания DAQ, использованные для любой кадр отражения, сохраняется в виде файла-приложения `.daq`. |

> **Примечание — значение по умолчанию отличается от значения, указанного в CLI.** По умолчанию `ArraySession.capture()` принимает значение `processing="debayered"`; по умолчанию команда `chloros-cli lattice array-capture` принимает значение `processing="all"`. Явно передайте `processing="all"` явно из файла SDK, чтобы отразить многоуровневое сохранение CLI /GUI.

### Режимы съёмки и устройства записи

Поверхность массива повторяет панель съёмки графического интерфейса: режимы «Одиночный» / «Непрерывный» / «Интервальный» / «Максимально быстрый затвор», а также два устройства записи (композитное видео в реальном времени и серия необработанных кадров → повторная обработка в автономном режиме).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**— это цикл «Непрерывный/Интервальный» в режиме «SDK». Поскольку нет команды `Ctrl+C` для прерывания этого цикла из скрипта, вы**обязаны** передать `count` и/или `duration_s` (цикл останавливается при достижении любого из них). `interval_s` измеряется с начала каждого прохода (в соответствии с графическим интерфейсом). Остальные kwargs передаются напрямую в `capture()`.
- **`record`** предназначен для *мониторинга*: он фиксирует отображаемый в реальном времени составной индекс в том виде, в котором он отображается, поэтому для поступления кадров должен быть открыт объединенный поток. Один рекордер составного индекса на каждый массив (генерирует исключение, если один уже запущен).
- **`burst` → `build_video`** предназначен для *анализа*: `burst` записывает необработанные кадры + манифест для каждого кадра + один файл `.daq` на каждое отдельное считывание DLS в рамках `<output>/bursts/<base>/` на полной скорости цикла захвата (без цепочки, без exiftool, без просмотра в реальном времени). `build_video` синхронизирует каждый кадр по времени с ближайшим `.daq` и повторно запускает цепочку обработки яркости/отражательной способности/индекса. `products` представляет собой список `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (по умолчанию: комбинированный индекс). `burst().stop()` также автоматически запускает построение комбинированного индекса с максимальной точностью, результат которого возвращается как `build_job` в результатах остановки.

#### `RecorderHandle`

Возвращается функциями `ArraySession.record()` и `ArraySession.burst()`. Используйте его в качестве контекстного менеджера для автоматической остановки при выходе из области действия или управляйте им вручную.

| Элемент | Описание |
| --- | --- |
| `job_id` | Идентификатор задания бэкэнда (строка). |
| `kind` | `"composite"` (из `record`) или `"raw"` (из `burst`). |
| `start_stats` | Словарь, возвращаемый вызовом `start`. |
| `result` | `None` во время работы; итоговый словарь результатов остановки после остановки. |
| `stats(timeout=10.0)` | Статистика задания в реальном времени (количество записанных кадров, фактическая частота кадров, прошедшее время). |
| `stop(timeout=60.0)` | Остановка записывающего устройства; возвращает и кэширует окончательный результат. Идемпотентно (повторный вызов возвращает результат из кэша). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Подключение к уже подключенному массиву — `attach_array`

Если массив уже запущен (его открыл графический интерфейс или предыдущий сеанс SDK вызвал `connect_array`), используйте `attach_array`, чтобы получить дескриптор к нему вместо повторного подключения. `connect_array` всегда возвращает ошибку «Камера  <sn>уже </sn>находится <sn>в массиве <id>», поскольку отправка запроса POST с кодом `/array/connect` для элемента в пуле не является идемпотентной; `attach_array` считывает `/api/camera/array/list` и сопоставляет их либо по array_id, либо по серийным номерам.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Шаблон: SDK скрипты, работающие совместно с настольным графическим интерфейсом, должны сначала пробовать `attach_array`, а если в пуле ещё нет массива, переключаться на `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Важно — выход из context-manager ДЕЙСТВИТЕЛЬНО приводит к разрыву соединения.**`ArraySession.disconnect()` всегда отправляет запрос POST на `/array/disconnect`; здесь нет защитного механизма «attached-not-принадлежности, как в случае с `CameraSession` / `DAQSensorSession`. Если вы используете совместное размещение с графическим интерфейсом и не хотите разрушать массив при выходе из области видимости,**не используйте блок `with`** — сохраните дескриптор в обычной переменной и пропустите явное использование `disconnect()`:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Помощник по анализу сети

Полезно перед открытием массива — позволяет прогнозировать, подойдут ли ваши предлагаемые настройки:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` является одним из `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (в противном случае — `error`). `auto_capped_fps` означает, что запрашиваемое разрешение подходит для кольца RX только при ограниченной частоте срабатывания триггера — сохраните разрешение и перейдите от `target_fps=result["recommended"]["recommended_target_fps"]` к `connect_array` (см. [Пример 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Как читать проекцию** (та же схема, что и на панели «Настройки массива» в графическом интерфейсе):

- **Серия снимков (`frame_bytes_total`) суммируется по каждой камере в реальном пиксельном формате каждой камеры.**Моно**M3M**камеры передают поток Mono12 (2 бит/пиксель) независимо от переданного значения `pixel_format`, поэтому кадр с полным разрешением от 4 камер составляет**~25 МБ** при использовании трёх монохромных камер, а не ~12,6 МБ, как следует из предположения о 8-битной глубине цвета всех камер. Бэкенд определяет формат каждой камеры по её модели.
- **Пропускная способность (`burst_fits_nic_ring`) учитывает скорость считывания**, а не соотношение полного пакета и кольца: режим «sim-emit» подходит, когда хост считывает данные из кольца RX быстрее, чем камеры его заполняют. Хост 10G + камеры 1 GbE**допускают** передачу в полном разрешении даже тогда, когда пакет превышает емкость кольца; хост 1 GbE блокирует (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` — это консервативный верхний предел последовательной выборки** — `max(readout+emit, N×emit)` с ограничением передачи данных на каждую камеру до скорости камерного соединения 1 GbE, независимое от экспозиции. Например, ~2,8 кадров в секунду для массива из 4 камер с полным разрешением и 12-битной глубиной цвета (соответствует измеренному в среде выполнения значению ~2,7–3,0). Полная модель: [CLI Ссылка → Модель частоты кадров и серийной съемки массива](cli-reference.md#array-fps--burst-model).
- **Переподписка (`oversubscribed: true`) означает, что нижний предел N × на камеру превышает безопасный с точки зрения коллизий верхний предел** — поля fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) принимают значение 0, и автоматическое сжатие/объединение не могут исправить ситуацию (они уменьшают количество байтов на кадр, а не количество байтов в секунду с фиксированной скоростью). Способами устранения проблемы являются уменьшение количества камер, использование джамбо-фреймов или более быстрая сетевая карта; `max_cams_collision_safe` сообщает о предельном значении (6 камер с полным разрешением по 1 Гбит/с при MTU 1500, 9 — с джамбо-кадрами). Ответ также содержит коды `aggregate_demand_bps`, `collision_safe_ceiling_bps` и `per_cam_floor_bps` (8 МБ/с). См. раздел [Переподписка](#over-subscription-the-per-cam-floor).

### Обнаружение и отображение списка

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Массивы LATTICE запускают непрерывную автоэкспозицию (AE) в фоновом режиме сразу после подключения, но для свеженаведённой сцены требуется некоторое время, чтобы достичь стабильности. **Smart-capture** — это готовое удобное решение: она опрашивает экспозицию каждой камеры, ждёт, пока массив не стабилизируется по всему окну, а затем запускает съёмку. Это эквивалентно действию в графическом интерфейсе: кнопка «smart-capture» в настольном приложении вызывает ту же конечную точку бэкэнда.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

При использовании режима `ChlorosProject` (следующий раздел) появляется больше настроек:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

По умолчанию политика интеллектуальной экспозиции (smart-AE) настраивается консервативно. Ужесточите настройку `exposure_tolerance_pct` для требовательных радиометрических задач; ослабьте — для быстро меняющихся сцен, где достаточно «приблизительного» результата.

---

## Сеансы датчиков DAQ

Постоянный пул бэкэндов для спектральных датчиков (DAQ-U через USB, DAQ-M через BLE, DAQ-E через Ethernet). Отражает работу камеры: интеллектуальное обнаружение, повторное использование пула, идемпотентное подключение.

### Интеллектуальное обнаружение (Zero-Config)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Приоритет: Ethernet → BLE → USB. Передайте любой явный указатель, чтобы зафиксировать транспорт.

### Зафиксированный транспорт

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Методы `DAQSensorSession`

| Метод | Описание |
| --- | --- |
| `status(timeout=10.0)` | Сводка записи пула (состояние потоковой передачи/записи, диапазон длин волн, SHA калибровки, время интегрирования, frame_avg, состояние AE). |
| `latest(n=1, timeout=10.0)` | Возвращает до N последних спектральных кадров. |
| `stream_start()` / `stream_stop()` | Возобновление / приостановка потоковой передачи (дескриптор остаётся открытым). |
| `record_start(output_dir=None, device_name=None)` | Запуск записи файла .daq. Возвращает путь к файлу. Отказ для DAQ-U/M без пакета калибровки AWS (DAQ-E исключается). |
| `record_stop()` | Остановка записи. Возвращает `{path, rows}`. |
| `disconnect()` | Освобождение из пула. Не выполняется для присоединённых, но не принадлежащих пользователю дескрипторов. |

> **Профили коррекции капа (`cap_id`) не являются регулятором SDK.** `connect_daq_sensor()` / `DAQSensorSession` не предоставляют доступ к параметру `cap_id` или методу `set_cap`. Выберите профиль ограничения паркапрофиль коррекции с помощью CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) или маршрутов `/api/daq` HTTP бэкенда (`/api/daq/connect` и `/api/daq/<id>/cap-id` принимают `cap_id`).

### Обнаружение — поиск адреса для подключения

`discover_daq_sensors()` сканирует USB / BLE / ETH в поисках датчиков, которые *можно* открыть. Это аналог `discover_lattice_cameras()` для DAQ и единственный способ получить **BLE MAC-адрес DAQ-M** — у DAQ-E есть имя хоста, а у DAQ-U — COM-порт, но MAC-адрес не указан ни на устройстве, ни в списке ОС.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Поле | Описание |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | COM-порт / MAC-адрес BLE / имя хоста — передать в `connect_daq_sensor` в виде `port=` / `mac=` / `eth_host=`. |
| `display` | Метка, понятная человеку. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` или `None` для порта, который сканирование не может идентифицировать (USB-последовательные адаптеры невозможно различить без зонда, поэтому неизвестные элементы отображаются, а не скрываются). |
| `extra` |сведения о транспорте (объявленное имя BLE, производитель USB, IP/прошивка DAQ-E и т. д.). Пустые значения опускаются. |

| Параметр | По умолчанию | Описание |
| --- | --- | --- |
| `transports` | все три | Последовательность (или строка в формате CSV), ограничивающая сканирование. Стоит передавать, если вы знаете, что хотите — BLE является самым медленным звеном. |
| `scan_timeout` | 5 | Окно сканирования для каждого транспорта в секундах; бэкенд ограничивает его значением 1–20. |
| `timeout` | 60,0 | Верхний предел HTTP для всего вызова (как и в других местах в SDK). |
| `auto_start_backend` | `True` | Запускает локальный бэкенд, если он не запущен. Никогда не запускается для удалённого `backend_url`. |

> **Датчики, уже открытые в пуле, не отображаются.** Подключённое периферийное устройство BLE прекращает рекламировать себя, а открытый COM-порт не может быть просканирован, поэтому список обнаружения содержит только то, что *доступно для подключения*. Пустой результат сразу после подключения устройства является ожидаемым — используйте `list_daq_sensors()` для того, что у вас уже есть. Транспорты, сканирование которых невозможно выполнить (не установлены bleak / zeroconf), пропускаются, а не вызывают ошибку, поэтому компьютер без Bluetooth по-прежнему получает ответы по USB и Ethernet.

### Пример

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Совместное использование с GUI / CLI

Если в GUI уже открыт датчик, вызов `connect_daq_sensor(port="COM3")` из Python возвращает дескриптор с меткой `already_connected=True`. Дескриптор `disconnect()` сеанса в этом случае является не выполняет никаких действий, поэтому ваш скрипт SDK не отключает датчик из графического интерфейса при выходе из программы.

### Классы прямого доступа к оборудованию (без бэкэнда)

`daq_sdk` повторно экспортируется `chloros_sdk`, поэтому вы также можете управлять датчиками от начала до конца в процессе работы без использования бэкэнда:

> **Доступность:**`daq_sdk` поставляется в составе настольной установки Chloros,**но не** в составе пакета PyPI — `pip install chloros-sdk` предоставляет вам `lattice_sdk`, но оставляет `chloros_sdk.DAQ_AVAILABLE == False`. Проверьте этот флаг перед использованием этих классов; на хост-диске, где установлен только pip, используйте вместо этого датчик через [`connect_daq_sensor()`](#daq-sensor-sessions), который не требует локальных транспортных библиотек.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Предпочтительно используйте путь с интеллектуальным подключением (`connect_daq_sensor`), если требуется совместное владение с графическим интерфейсом; используйте классы прямого доступа для скриптов без интерфейса, которые владеют датчиком исключительно.

---

## Автоматизация проектов — `ChlorosProject`

Сохраненный проект «Chloros» представляет собой папку, содержащую `cameras.json` + `sensors.json` + `project.json`. `open_project` загружает манифест, а `connect_all` переводит все сохраненные устройства в режим онлайн с их сохраненными настройками — в том же состоянии оборудования, которое было бы создано с помощью графического интерфейса.

### Минимальный пример

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Или в качестве менеджера контекста:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Методы `ChlorosProject`

| Метод | Описание |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Обнаруживает и подключается ко всем сохраненным устройствам. Возвращает отчет о подключении по классам. Использует запущенный бэкенд, если таковой прослушивает на `127.0.0.1:5000`; в противном случае без предупреждения переключается на прямое (без бэкенда) управление устройствами `lattice_sdk` — никогда не запускает бэкенд. |
| `disconnect_all()` | Разрывает все соединения. |
| `capture_all(output_dir=".")` | Один кадр с каждой камеры + массив + спектр с каждого датчика. |
| `stream(camera, overlays=False, fps=10.0)` | Генератор, выдающий BGR-кадры `numpy` с указанной камеры (или массива). `overlays=False` представляет собой прямой цикл захвата `lattice_sdk` (массивы выдают словари `{serial: frame}`). `overlays=True` проходит через `ChlorosLocal.camera_stream()` → канал MJPEG бэкенда `/api/camera/<serial>/stream-annotated`, при этом сохраненный блок камеры `ui.overlay`X, передаётся в качестве параметров запроса. Требует режима бэкэнда и **автономной камеры**: камера в прямом режиме вызывает ошибку `RuntimeError` (бэкенд не может получить камеру, принадлежащую этому процессу), а массив вызывает ошибку `NotImplementedError` (наложение композитного изображения для каждого члена — передача элемента по имени). Однократный эквивалент: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Запустить выравнивание для каждого подключённого в данный момент массива. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Запустить калибровочный / индексации для изображений проекта (обертывает `ChlorosLocal.process`; эти четыре — **единственные** допустимые аргументы — `indices=` и т. д. вызывают исключение `TypeError`; задание индексов осуществляется с помощью `ChlorosLocal.configure()`). Лениво создаёт `ChlorosLocal()`, который автоматически запускает бэкэнд. |

Атрибуты:
- `proj.cameras` — `Dict[str, CameraHandle]` с ключом по имени И серийному номеру.
- `proj.arrays` — `Dict[str, ArrayHandle]` с ключами по имени И array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` с ключами по имени и slot_id.
- `proj.config` — `project.json["config"]` (словарь).

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Уровни обработки.** `capture()`, `grab()` и `frame_stream()` все принимают один и тот же токен `processing`,
а цепочка является кумулятивной — каждый уровень выполняет всё, что находится выше него:

| Уровень | Результат | Примечания |
| --- | --- | --- |
| `raw` | 1-канальный Байер, нативный для датчика | Без демозаики. Наложения на этом уровне недоступны. |
| `debayered` | 3-канальный BGR (**по умолчанию**) | Билинейная демозаика. Единственный уровень, работающий без режима бэкенда. |
| `radiance` | float32, Вт/м²/ср/нм | Полная радиометрическая цепочка: демозаика + 3×3 разделение (мультиспектральное) + DSNU + плоское поле + шкала NIST, с выделением экспозиции и коэффициента усиления, чтобы значения были абсолютными. |
| `reflectance` | uint16, 32768 = 1,0 | Яркость, деленная на интенсивность нисходящего излучения (ρ = π·L/E). Требуется показание DLS/DAQ — см. примечание ниже. |
| `display` | 8-бит, приближенный к sRGB | Рендеринг, эквивалентный GUI: CCM + баланс белого + гамма через активный цветовой профиль камеры. |

Все, что отличается от `debayered`, требует режима бэкенда; камера в прямом режиме генерирует
`NotImplementedError`. `reflectance` требует пригодного показателя интенсивности излучения вниз — конечная точка кадра автоматически
автоматически помещает сгруппированные данные DAQ в слот DLS камеры, но без привязки к DAQ цепочка отклоняет
выход отражательной способности и честно отмечает понижение качества в возвращаемых метаданных, а не молча
возвращает продукт более низкого качества.

> **Шкала DN отражательной способности — не задавайте её жестко.** Отражательная способность LATTICE использует `32768` = ρ 1,0 и помечает
> XMP-тег `Chloros:PixelScale=32768`; отражательная способность Survey3 использует `65535` = ρ 1,0 и не содержит
> тегов `Chloros:*`. Прочитайте тег и разделите на него. Оно определено в области uint16, поэтому остаётся
> `32768` для каждого формата, который изменяет масштаб (16-битовый TIFF, 8-битовый PNG /JPG, 32-битовый в процентах) — сначала нормализуйте
> сохраненный тип данных обратно в uint16 (×257 из 8-битного, ×65535 из float). Единственное исключение:
> захват с 8-битным источником, записанный как 8-битный TIFF, *обрезается*, а не масштабируется, поэтому никакой масштаб не описывает
> его — Chloros в этом случае полностью опускает `PixelScale` и кортеж MicaSense. Отравление, отсутствующий
> тег в файле отражательной способности LATTICE, следует рассматривать как «отсутствие действительного масштаба», а не как значение по умолчанию.

> **EXIF переносится в экспорт.** `process()` копирует блок GPS исходного снимка
> **и его ExifIFD** в каждый продукт, поэтому экспортные файлы содержат `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` и `CameraSerialNumber`, а также
> геореференцию. На основании кода `FocalLength` Pix4D вычисляет расстояние между точками съёмки — без него
> реконструкция выполняется в совершенно неверном масштабе (в одном из измеренных случаев площадка размером 411 м
> превратилась в площадку размером 47,8 км). Копия намеренно не является `-all:all`: структурные теги IFD0 нарушают
> вывод LATTICE, а `ExifImageWidth`/`Height` исключены, поскольку они описывают исходную
> съёмку, а не экспортированный растр.

Подфлаги этапа захвата (применяются к радиометрическим уровням — `radiance`, `reflectance`, `display`):

| Флаг | По умолчанию | Значение |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + плоское поле + 3x3 разложение + радиометрическая шкала NIST. |
| `apply_white_balance` | `True` | LUT для баланса белого. Поддержка DLS, если к камере подключен модуль сбора данных (DAQ). |
| `apply_index` | `False` | Оценка растительного индекса. |
| `index_expression` | `None` | Переопределение формулы. Непустое значение → автоматически включает индекс. |
| `annotated` | `False` | Наложение элементов интерфейса (зебра/сетка/пики). Недоступно для `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **Тип возвращаемого значения — `CapturePathMap`, а не `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` — это `Dict[str, Union[str, List[str]]]`: одноуровневый
> `processing` предоставляет каждому серийному номеру один путь, тогда как многоуровневый (`"all"` или
> явный список `levels`) предоставляет **упорядоченный список** всех продуктов, сохраненных для этой
> камеры. Комбинированный композит в реальном времени, если таковой транслируется, поступает под дополнительным
> ключом `"combined"`, а не под серийным номером. Код, предполагающий использование `str`, выдает ошибку при работе со
> списковой формой, причём никакой проверщик типов не выдает предупреждения — аннотация указывала `Dict[str, str]`
> в течение некоторого времени после выпуска формы списка, именно поэтому и существует этот псевдоним. Нормализуйте
> при необходимости использования плоской формы:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Выравнивание массивов

`ArrayHandle` предоставляет полную поверхность выравнивания. По умолчанию профили действительны только в рамках сеанса — вызовите `export_alignment()` явно, чтобы сохранить их.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Выравнивание при подключении

`connect_all(align=...)` может автоматически выравнивать каждый массив при подключении:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Если не указано иное, используется `project.json["config"]["auto_align_on_connect"]`.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Прямое аппаратное обеспечение (без бэкэнда)

Если требуется полное отсутствие зависимости от бэкэнда (CI, роботы без интерфейса, встраиваемые системы), импортируйте `lattice_sdk` и `daq_sdk` напрямую — оба они реэкспортируются `chloros_sdk`. Проверка на `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: `lattice_sdk` входит в пакет PyPI (но требует наличия среды выполнения Arena SDK), тогда как `daq_sdk` поставляется только с настольной установкой.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Пресеты и триггер

Три из четырёх пресетов **freeзапуска**: камера ведёт непрерывную съёмку, и
`capture()` возвращает следующий кадр. `triggered` является исключением — он активирует
камеру на аппаратный фронт по линии 2, поэтому она ничего не снимает, пока такой фронт не появится.

| Пресет | Триггер | Использовать, когда |
| --- | --- | --- |
| `default` | free-run | общее использование |
| `high_speed` | free-run | 8 бит, ограничение 60 кадров/с, короткая экспозиция |
| `high_quality` | свободный режим | 12-битная, без ограничения fps — обычный выбор для фотосъёмки |
| `triggered` | **в режиме ожидания, линия 2** | камера подключена к синхронизационному кабелю M8, и её запускает что-то другое |

Если вы выберете `triggered` (или самостоятельно настроите `trigger_mode="On"`) без
управления линией 2, каждый `capture()` завершится таймаутом — что правильно, поскольку вы попросили
камеру подождать. В «SDK» объясняется, что происходит в этом случае; см.
[SC_ERR_TIMEOUT во время съёмки](#direct-hardware-backend-free).

> **Примечание — сообщения «GVSP probe» / `SC_ERR_TIMEOUT -1011` при подключении не являются ошибками.**&gt; При подключении SDK пытается согласовать**джамбо-фреймы** (пакеты GVSP размером 9000 байт) для повышения пропускной способности. На прямом соединении «точка-точка» между сетевыми картами (например, с локальным адресом канала `169.254.x.x`) сеть обычно не поддерживает передачу джамбо-фреймов, поэтому этот тест завершается с превышением времени ожидания и в журнале появляются такие строки:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Это **предусмотренный резервный вариант**: сетевой адаптер SDK автоматически переключается на стандартные пакеты размером 1500 байт, и камера продолжает подключаться в обычном режиме (следующие строки `[chunk-enable …]` являются частью обычной последовательности подключения). Захват данных по-прежнему работает.
>
> Вы можете пропустить этот тест, но **он не просто подавляет запись в журнал — он отключает джамбо-фреймы.** Камера отвечает на пинги с флагом «Don&#x27;t-Fragment» только до 1500 байт, независимо от качества вашей сети, поэтому один только тест пинга никогда не сможет обнаружить джамбо-фреймы; этот тест — единственное средство, способное это сделать. Отключите его, и камера будет использовать стандартные пакеты размером 1500 байт бесконечно, в любой сети:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Это имеет смысл только в той сети, о которой вы *знаете*, что она не поддерживает джамбо-фреймы, где это экономит примерно одну секунду времени подключения на каждую камеру. Поскольку это реальный компромисс, а не просто косметическое изменение, теперь при его использовании в файле «SDK» появляется соответствующее сообщение:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Не трогайте эту настройку, если у вас нет на то причины.** Если оставить эту настройку включённой, при каждом подключении будет заново определяться реальные характеристики вашей сети: подключитесь к коммутатору, поддерживающему пакеты jumbo, и при следующем подключении пакеты jumbo будут поддерживаться автоматически, без необходимости настройки и перезапуска.
>
> Если вы *хотите* пропускную способность пакетов jumbo, включите поддержку пакетов jumbo по всей длине трасса (MTU сетевой карты 9000 + коммутатор, пропускающий такие пакеты), либо зафиксируйте размер с помощью `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000`, если знаете, что канал поддерживает его — хотя лучше использовать параметр `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` для отдельных команд, чем устанавливать его постоянно, поскольку фиксированный размер пропускает тестирование и не позволяет адаптироваться к сети, расположенной впереди. **Каждое** устройство на пути должно пропускать пакеты jumbo — включая любые PoE-разветвители или инжекторы, которые обычно являются причиной того, что дажене может передавать пакеты jumbo.

> **`SC_ERR_TIMEOUT -1011` во время `capture()` / `grab*()` — это другая проблема; в данном случае речь идет о настоящей ошибке.**&gt; Примечание выше касается только ошибки `-1011`, зарегистрированной**зондом connect-time**. Та же самая ошибка, возникшая при**захвате**, означает, что камера подключилась нормально, но не отправляет изображения:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> Признаком этой проблемы является камера, у которой канал *управления* исправен — обнаружение работает, настройки и записи `[chunk-enable …]` выполняются успешно — в то время как *каждый* кадр превышает таймаут.
>
> **Обычно это происходит из-за того, что камера настроена на аппаратный триггер.** При ошибках `trigger_mode="On"` и `trigger_source="Line2"` камера не передаёт ничего, пока на синхронизационном кабеле M8 не появится электрический фронт. Если кабель, управляющий этой линией, отсутствует, каждый захват будет ждать бесконечно. Камера не сломана, и сеть работает нормально — она делает именно то, что ей было предписано.
>
> Коды `CameraSettings()` и `default` / `high_speed` / `high_quality` задают режим свободного работы, и захват, у которого истек таймаут в режиме готовности, выдает соответствующее объяснение вместо простого сообщения `-1011`. `PRESETS["triggered"]` по замыслу активирует Line2, что предусмотрено конструкцией.
>
> Чтобы принудительно перевести любую камеру в режим свободного хода:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Если при использовании `trigger_mode="Off"` тайм-аут по-прежнему возникает, это означает, что камера действительно не передает данные — пришлите нам лог и `ip link show`.

#### Цветовые профили (предварительный просмотр в реальном времени RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` выбирает цветовой профиль дисплея для **предварительного просмотра** на камерах RGB (мультиспектральные камеры игнорируют этот параметр):

| Профиль | Значение |
| --- | --- |
| `raw` | Полностью обойти радиометрическую цепочку. |
| `linear` | DSNU + flat + WB, без CCM, без гаммы. |
| `natural` | Линейная коррекция + измеренный CCM + гамма sRGB, только с «дешёвой» обработкой (сглаживание цветности + десатурация светлых участков) — реалистичный вариант по умолчанию. |
| `enhanced` | `natural` плюс полная обработка с использованием технологии Hub-Parity (удаление ореолов, яркость, локальный контраст CLAHE). Более насыщенный вид при примерно **двойной стоимости обработки каждого кадра**, а значит, более низкая частота кадров в режиме LIVE. |
| `custom_temp` | `natural`, но баланс белого привязан к `custom_cct_k` Кельвина (DLS игнорируется; ограничено 2000–10000 K на стороне бэкэнда). |

Профиль является регулятором скорости/визуального эффекта **только для предварительного просмотра в реальном времени**: сохраненные снимки всегда получают полную насыщенную обработку независимо от выбранного профиля, поэтому выбор `natural` с целью экономии времени обработки кадра не снижает качество того, что записывается на диск. Неизвестный профиль повышает значение `ValueError`; когда бэкенд chloros доступен, изменение также отправляется ему посредством POST, так что следующий кадр предварительного просмотра отражает его (пользователи режима direct-SDK, не имеющие бэкенда, всё равно получают изменение настроек).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Монохромные (M3M) камеры и `Calibration`

Монохромная камера **M3M** (`M3M-<lens>-F<wavelength>`) является однополосной: одна плоскость оттенков серого, отсутствует мозаика Байера, без спектральной матрицы перекрестных помех 3×3. `Calibration` распознаёт её и выставляет флаг `is_mono`. Коэффициент отражения по-прежнему применяется в качестве радиометрической карты для каждого диапазона (матрица разложения — единичная), но многополосные вычисления на одной камере дают реальные результаты, а не бессмысленные:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Чтобы построить индекс растительности с помощью монохроматического оборудования, объедините несколько камер M3M, работающих на разных длинах волн, в выровненный многополосный стек (см. [Выравнивание массива](#array-alignment)) и вычисляйте индекс по всему этому стеку, а не по одной камере.

Прямой режим DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **`apply_sensor_settings` допустимые ключи**— именно `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; устарело, заменено на `cap_id`), `filter_model` (DAQ-M) и `cap_id` (все типы DAQ; `None`/`""`/`"none"` = простой датчик, без коррекции емкости). Неизвестные ключи**тихо игнорируются** — например, `{"integration_time": 64}` ничего не делает (должно быть `integration_time_ms`). Возвращает `{"applied": [...], "errors": {...}}` и никогда не генерирует исключение.

`chloros_sdk` повторно экспортирует только базовую поверхность, использованную выше. Полный публичный API `daq_sdk` (22 имени) добавляет следующее — импортируйте их напрямую из `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Исключения

Перехватить базовый класс для обработки «любых сбоев в работе Chloros»:

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

> `ChlorosAuthenticationError` и `ChlorosConfigurationError` экспортируются на верхнем уровне наряду с остальными; их также можно импортировать из `chloros_sdk.exceptions`, как показано.

Иерархия:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Примеры от начала до конца

### 1. Обработка папки с настраиваемой полосой прогресса

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Массив Live LATTICE → Коэффициент отражения + эталон DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Кампания захвата данных на основе проекта

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Поток кадров с нескольких камер → конвейер NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Скрипт сбора данных без интерфейса пользователя напрямую с аппаратного уровня (без бэкэнда)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Проверка возможностей перед подключением массива из 4 камер

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Эквивалент рецепта захвата (чистый Python)

DSL рецептов CLI имеет прямой эквивалент в Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Автозапуск бэкэнда

Точки входа smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` и `discover_lattice_cameras` — представляют собой «тонкие» клиенты HTTP, которые предполагают, что бэкенд прослушивает порт `127.0.0.1:5000` (URL по умолчанию для интерфейса Smart-Connect). Если графический интерфейс или CLI уже запущены, то один из них уже работает. При запуске из простого скрипта может оказаться, что — поэтому эти функции **автоматически запускают входящий в комплект бинарник бэкэнда** (без оконного интерфейса, так же, как это делает `ChlorosLocal`) перед своим первым вызовом, а затем ждут до `backend_startup_timeout`, пока он не запустится.

Правила:

- **Запускается только локальный URL.** `backend_url`, указывающий на `localhost` / `127.0.0.1` / `[::1]` является допустимым; любой другой хост считается чужой машиной и никогда не запускается.
- **Бэкенд остаётся запущенным для повторного использования** (так же, как и в случае с CLI) — при завершении работы вашего скрипта не происходит неявного завершения работы бэкенда. При повторном запуске скрипта используется тот же действующий бэкенд.
- **Отключите эту функцию с помощью `auto_start_backend=False`** при любом из этих вызовов (например, если вы указали удалённый бэкенд или самостоятельно управляете жизненным циклом бэкенда).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Если входящий в комплект бинарный файл не удаётся найти или запустить, последующий вызов HTTP генерирует трассировку, требующую действий, **зависимую от платформы** ошибку `ChlorosConnectError` вместо простого сообщения об отказе в подключении — на сайте Windows она укажет вам на настольное приложение или команду `chloros-cli`; на сайте Linux (без графического интерфейса) он указывает на команду `chloros-cli` или на `.deb`.

---

## Среда и заголовки

SDK помечает каждый вызов бэкэнда HTTP с помощью `X-Chloros-Client: sdk`. Бэкэнд применяет правила лицензирования SDK / CLI (требуется вход **и** требуется платный тарифный план Chloros+), а не бесплатный тарифный план для графического интерфейса. Это настраивается автоматически при импорте — вам не нужно ничего делать.

`http://localhost` и `http://127.0.0.1` распознаются как локальный бэкенд. Вызовы других хостов (например, вашего собственного аналитического сервиса) остаются без изменений.

Переопределите бэкенд URL, указав `backend_url=` (или `api_url=` на `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(`backend_url` без loopback достигает только бэкенда source/dev — поставляемые бэкенды привязываются только к loopback; см. раздел «Режим удаленного бэкенда» для схемы туннелирования.)

---

## Управление версиями и совместимость

- Версия SDK предоставляется как `chloros_sdk.__version__`.
- SDK привязывает поведение к версии поставляемого бэкэнда. Совместное использование более старого SDK с более новым бэкэндом обычно работает (конечные точки, совместимые с более поздними версиями), но совместное использование более нового SDK со старым бэкэндом может привести к появлению ошибок `404` на новых конечных точках — обновите настольное приложение, чтобы обеспечить совместимость.
- Поверхность smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) и конечная точка анализа сети возвращают стабильные схемы JSON; новые поля добавляются.

---

## Рекомендации по устранению неполадок

- **`ChlorosAuthenticationError: Login required`** → Запустите `chloros-cli login EMAIL PASSWORD` один раз на этом компьютере или войдите в систему через настольное приложение Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → Вызовы Smart-Connect автоматически запускают локальный бэкенд, поэтому эта ошибка появляется только в том случае, если не удаётся найти или запустить входящий в комплект бинарный файл (например, на хосте, где установлен только pip и отсутствует пакет для рабочего стола). Сообщение зависит от платформы: на Windows откройте приложение для рабочего стола или выполните любую команду `chloros-cli`; на Linux выполните команду `chloros-cli` (графический интерфейс отсутствует) или установите `.deb`. Для удалённого бэкенда передайте `backend_url=` (и `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** при импорте → `lattice_sdk` не удалось загрузить (как правило, не установлены DLL-библиотеки среды выполнения Arena SDK). Не-поверхность без камеры по-прежнему работает.
- **Array connect возвращает разрешение ниже нативного**→ Функция smart-prep бэкэнда автоматически уменьшает размер кадра, чтобы он поместился в канал передачи данных. Используйте `analyze_array_network()`, чтобы выяснить причину, после чего либо обновите соединение, либо примите уменьшение, либо передайте `force_tier="slip-emit-and-capture"` для последовательной съемки. Механизм защиты от уменьшения размера**не** распространяется на совокупную переподписку (`oversubscribed: true`, поля fps равны 0): проблему избыточного количества камер для канала связи невозможно устранить с помощью биннинга или ROI — уменьшите количество камер, включите джамбо-фреймы или перейдите на более быструю сетевую карту (см. [Переподписка](#over-subscription-минимального значения на камеру)).
- **`analyze_array_network()` сообщает, что кольцо приёма сетевой карты очень мало (~0,26 МБ) / шлюзы соединения с сообщением «FRAMES WILL DROP»** → Кольцо приёма сетевой карты хоста находится в состоянии по умолчанию (часто сбрасывается на 32 после обновления драйвера сетевой карты). На адаптере Realtek USB 10GbE установите `ReceiveBufferLen=256` и `PendingReceives=64` (с повышенными правами), а затем перезапустите бэкенд, чтобы он заново прочитал кольцо. Полная процедура: [CLI Справочник → Настройка и оптимизация сетевой карты хоста](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Хост зависает при перезапуске/выключении, позже возникают ошибки WMI `Invalid class` / сетевая карта не включается** → Устаревший драйвер USB 10GbE вызывает `DRIVER_POWER_STATE_FAILURE` (синий экран смерти (BSOD) `0x9F`). Обновите драйвер адаптера до актуальной версии (≥ 2026) и заново примените настройки кольца приема. См. [Справочник по CLI → Настройка и оптимизация сетевой карты хоста](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Отражение отклонено** → Для получения отражения в абсолютной шкале к камере (или матрице) должен быть привязан активный модуль сбора данных (DAQ). Выполните привязку через графический интерфейс или используйте `processing="radiance"` (Вт/м²/ср/нм), который не требует сопряжённого датчика.
- **`smart=True` занимает больше времени, чем ожидалось** → Сходимость AE зависит от динамики сцены; увеличьте значение `exposure_tolerance_pct` или уменьшите значение `stability_window_s`, если требуется более быстрый (менее стабильный) триггер.

---

## См. также

- [Справочник по CLI](cli-reference.md) — каждая подкоманда CLI соответствует вызову SDK.
- [Руководство по датчикам DAQ](../daq/README.md) — правила подключения, калибровки и записи данных для конкретных датчиков.
- Онлайн-документация: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
