# API : Python SDK

{% hint style="info" %}
**Ищете полную версию API?** Эта страница представляет собой практическое руководство. Все общедоступные классы, методы, точные сигнатуры и примеры, готовые к копированию и вставке, находятся в [Справочнике по SDK](reference/sdk-reference.md), который оптимизирован для ИИ-помощников.**Работаете с ИИ-помощником?** Вставьте этот URL в чат, чтобы у него был полный актуальный Chloros 1.2.0 API:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Каждая страница этого руководства доступна в виде исходного кода Markdown по адресу, состоящему из её слэг-имени в нижнем регистре + `.md`, а всё руководство индексировано по адресу `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

**Chloros Python SDK** (`chloros-sdk` на PyPI) управляет всеми функциями настольного приложения, начиная с Python: пакетная обработка изображений, управление камерой LATTICE и массивом в режиме реального времени, сеансы сбора данных с световых датчиков, а также автоматизация сохраненных проектов. Это тонкий слой, наложенный на тот же локальный бэкенд, который используют графический интерфейс и CLI (HTTP на `127.0.0.1:5000`), поэтому поведение одинаково на всех трёх платформах.

## Установка

Установка состоит из двух этапов: сначала устанавливается пакет Chloros для рабочего стола (он предоставляет бэкэнд обработки и среды выполнения для аппаратного обеспечения), а затем — пакет Python.

**Шаг 1 — Установите Chloros.** Windows: запустите установщик для настольных систем (путь по умолчанию — `C:\Program Files\MAPIR\Chloros\`) со страницы [Скачать](download.md). Linux: установите пакет `.deb` ([Установка Linux](linux/linux-installation.md)).**Шаг 2 — Установите SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Возможно, вам даже не понадобится pip: каждый установщик поставляется с соответствующим колесом SDK. Установщик Windows автоматически устанавливает его в системный репозиторий Python; установщик Linux `.deb` помещает его в `/usr/lib/chloros/sdk/` и выводит точную команду `pip install --user`. PyPI обновляется при выпуске новых версий, поэтому `pip install chloros-sdk` соответствует последней стабильной версии.

**Шаг 3 — Войдите в систему один раз на каждом компьютере:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

Учетные данные кэшируются в `~/.chloros/` (на обеих платформах). На Windows вы можете аналогичным образом войти в систему через вкладку «Пользователь» (User) <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> в настольном приложении. Для SDK требуется платный тарифный план Chloros+ — см. [Требования к лицензии](#license-requirement) ниже.

| Требование | Подробности |
| --- | --- |
| **Установлен Chloros** | Windows: установщик для настольных компьютеров; Linux: пакет `.deb` (содержит бинарный файл бэкэнда) |
| **Python** | 3.7 или выше (разработано/протестировано на версии 3.10) |
| **Операционная система** | Windows 10/11 64-разрядная, Ubuntu 22.04 LTS или более поздние версии, либо NVIDIA Jetson (JetPack 6) |
| **Лицензия** | Активная учетная запись Chloros+, любой платный тариф (Copper или выше) |

## Победа за 60 секунд

Один вызов создает проект, импортирует папку, настраивает обработку и запускает конвейер — автоматически запуская бэкенд, если он ещё не работает:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(В Linux используйте пути Linux: `/home/user/drone_images/flight001`. Команда SDK работает одинаково на обеих платформах.)

Обрабатываете папку снимков LATTICE? Используйте оболочку, оптимизированную для LATTICE — она применяет правильные настройки по умолчанию (без определения целевой панели, стандартный дебейер):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — полный контроль над конвейером

Для любых задач, выходящих за рамки однострочной команды, используйте `ChlorosLocal`. Он запускает бэкэнд при первом использовании (`auto_start_backend=True`), создаёт и настраивает проекты, отслеживает ход выполнения и возвращает сводку по итогам выполнения.

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
Сохраняйте значение по умолчанию `http://127.0.0.1:5000` вместо замены на `localhost` — при использовании Windows `localhost` сначала преобразуется в `::1` и занимает ~2 секунды на каждый запрос при работе с бэкендом, поддерживающим только IPv4.
{% endhint %}

Используйте его в качестве контекстного менеджера для гарантированной очистки:

```python
import chloros_sdk

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

`configure()` принимает следующие ключевые слова: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` и `custom_settings`. Основные значения:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Регуляторы, специфичные для LATTICE (`input_level`, `radiometric_output`, семейство `array_alignment*`) описаны с полными таблицами значений в [Справочнике по SDK](reference/sdk-reference.md#supported-values).

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

По завершении `process()` присоединяет сводку обработки бэкенда в виде файла `result["summary"]`. Каждая запись в `summary["hints"]` представляет собой полное предложение, объясняющее всё, что заслуживает внимания — например, почему запуск не дал результата — и каждое предупреждение также повторно выводится в виде Python `UserWarning`, поэтому пустые запуски диагностируются автоматически, даже если вы никогда не просматриваете словарь:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` не генерируется, когда запуск не создает изображений.** Это единственное место, где SDK и CLI намеренно различаются: `chloros-cli process` рассматривает ситуацию «были запрошены результаты, но ни один не был записан» как сбой и завершает работу с ненулевым кодом, тогда как SDK возвращается в нормальном режиме и сообщает об этом состоянии через `summary` / подсказки. Если ваш конвейер должен останавливаться при пустом запуске, проверяйте это самостоятельно — просматривайте `summary` (или подсчитывайте файлы в папке проекта), а не полагайтесь на исключение.
{% endhint %}

## Smart Connect — аппаратное обеспечение в режиме реального времени

Три вспомогательных процесса открывают постоянные сессии в пуле оборудования бэкэнда — том же пуле, который использует графический интерфейс, поэтому скрипты SDK сосуществуют с настольным приложением, не конкурируя за последовательные порты или пропускную способность сети. Все три автоматически запускают локальный бэкенд, если он не запущен.

### Одиночная камера LATTICE — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Синхронизированный массив — `connect_array`

`connect_array` является рекомендуемой отправной точкой для многокамерных установок. Он запускает тот же поток интеллектуальной подготовки, что и графический интерфейс пользователя: анализ сети, автоматический выбор уровня синхронизации, синхронизацию времени по протоколу PTP, выбор формата пикселей для каждой камеры, инициализацию автоэкспозиции и подготовку триггера GPIO к срабатыванию. **Первый устройство в последовательной цепочке является ведущим** (оно генерирует импульс аппаратного триггера); остальные — ведомыми.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Добавьте `smart=True` к любой съемке массива, чтобы до запуска выжидать стабилизации автоматической экспозиции на всех камерах. Информацию о режимах съёмки (одиночный / непрерывный / интервальный / самый быстрый), регистраторах, режиме «пакетная съемка в видео» и выравнивании массива см. в [Справочнике по SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Датчик освещенности DAQ — `connect_daq_sensor`

Без аргументов функция `connect_daq_sensor()` автоматически определяет транспортный протокол (по приоритету: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Каждый фрейм содержит значение 135-точечного датчика `spectrum` (Вт/м²/нм после калибровки), флаг `is_saturated` и CIE `x`, `y`, `z`. Чтобы закрепить конкретный датчик или транспорт — надёжный выбор на хостах с несколькими сетевыми интерфейсами, где автообнаружение Ethernet может не обнаружить исправный DAQ-E с первой попытки — передайте один явный подсказку:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Обратите внимание, что профили коррекции верхнего регистра (`cap_id`) **не** являются параметром SDK — выбирайте их вместо этого с помощью `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Сохраненные проекты — `open_project`

Сохраненный проект Chloros сохраняет настройки подключенного оборудования (`cameras.json` + `sensors.json` наряду с `project.json`), а `chloros_sdk.open_project(path)` может одновременно восстановить все подключения и управлять захватом данных по именам устройств. См. раздел [Автоматизация проектов](reference/sdk-reference.md#project-automation--chlorosproject) в справочнике.

## Что получает установка, выполненная только через pip

Перед использованием аппаратных поверхностей проверьте флаги доступности на уровне модулей:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

На хосте, где установлен **только** `pip install chloros-sdk` и отсутствует пакет рабочего стола Chloros:

* `ChlorosLocal`, `process_folder` и `process_lattice_capture` **не** работают — им требуется бинарный файл бэкэнда, поставляемый в составе установщика рабочего стола.
* Помощники smart-connect (`connect_camera`, `connect_array`, `connect_daq_sensor`) являются чистыми клиентами HTTP, поэтому они работают с бэкендом на другом компьютере — однако поставляемые бэкенды привязаны только к лупбэку, поэтому вам необходимо самостоятельно настроить перенаправление порта (например, `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) и передать `backend_url="http://127.0.0.1:5000"` вместе с `auto_start_backend=False`. См. [Режим удалённого бэкэнда](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* Классы LATTICE, напрямую связанные с аппаратным обеспечением (`LatticeCamera`, `CameraPool`, …) импортируются, но для них требуется среда выполнения Arena SDK из пакета для настольных компьютеров — без неё `CAMERA_AVAILABLE` будет работать как `False`.
* `daq_sdk` (классы прямого сбора данных) поставляется с настольной установкой, а не с пакетом PyPI, поэтому `DAQ_AVAILABLE` на хосте, где установлен только pip, соответствует `False` — вместо этого управляйте датчиками DAQ через `connect_daq_sensor()`, подключаясь к (туннелированному) бэкенду.

## Требования к лицензии

Для доступа к SDK требуется активная учетная запись Chloros+ на любом платном тарифе — **Copper или выше**(Copper / Bronze / Silver / Gold); бесплатный тарифный план «Iron» не предоставляет доступа к SDK/CLI. Проверка осуществляется**на стороне сервера**: каждый запрос SDK должен сопровождаться как активной сессией, так и платным тарифом, в противном случае бэкенд возвращает `403` / `PLAN_UPGRADE_REQUIRED` (генерируемый как `ChlorosLicenseError` функцией `ChlorosLocal` и как `ChlorosConnectError` с помощью `connect_*`). Вызывающий, выйдя из системы, получает `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — повторный запуск `chloros-cli login` устраняет первый случай, но не второй.

Использование в автономном режиме работает в течение льготного периода тарифного плана: уровень доступа считывается из кэша проверки сервера (5 минут) или из кэша подписанных, привязанных к компьютеру лицензий (30 дней для месячных тарифных планов; до истечения срока подписки для годовых). По истечении льготного периода тариф переходит в бесплатный режим, и доступ к SDK блокируется до тех пор, пока устройство не установит соединение с сервером хотя бы один раз. Код ошибки `chloros-cli status` остается доступным в бесплатном тарифе, поэтому причина всегда видна. См. [Chloros+ Вход](chloros+-login.md).

## Исключения

Обработайте базовый класс для обработки «любых сбоев Chloros»:

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

* [Справочник по SDK](reference/sdk-reference.md) — полный набор поверхностей API, оптимизированный для ИИ-помощников.
* [Справочник CLI](reference/cli-reference.md) — каждая подкоманда CLI соответствует вызову SDK.
* [Скачать](download.md) — установщики для Windows и Linux.
