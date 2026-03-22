# API : Python SDK

**Chloros Python SDK** обеспечивает программный доступ к движку обработки изображений Chloros, позволяя автоматизировать процессы, создавать пользовательские рабочие потоки и легко интегрировать его с вашими приложениями Python и исследовательскими конвейерами.

### Основные особенности

* 🐍 **Нативный Python** — чистый, написанный на Python код для обработки изображений
* 🔧 **Полный доступ к API** — полный контроль над обработкой Chloros
* 🚀 **Автоматизация** — создание пользовательских рабочих процессов пакетной обработки
* 🔗 **Интеграция** — встраивание Chloros в существующие приложения Python
* 📊 **Готовность к научным исследованиям** — идеально подходит для научных аналитических конвейеров
* ⚡ **Параллельная обработка** — масштабируется в соответствии с количеством ядер вашего процессора (Chloros+)

### Требования

| Требование          | Подробности                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros установлен** | Windows: установщик для настольных компьютеров; Linux: пакет `.deb`                  |
| **Лицензия**          | Chloros+ ([требуется платный тарифный план](https://cloud.mapir.camera/pricing)) |
| **Операционная система** | Windows 10/11 (64-битная), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 или выше                                                |
| **Память**           | Минимум 8 ГБ ОЗУ (рекомендуется 16 ГБ)                                  |
| **Интернет**         | Требуется для активации лицензии                                     |

{% hint style="warning" %}
**Требования к лицензии**: Для доступа к Python SDK требуется платная подписка Chloros+ для API. Стандартные (бесплатные) тарифные планы не предоставляют доступ к API/SDK. Перейдите по ссылке [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), чтобы перейти на более дорогой тарифный план.
{% endhint %}

## Быстрый старт

### Установка

Установите через pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Первоначальная настройка**: Перед использованием SDK активируйте лицензию Chloros+, открыв Chloros, Chloros (браузер) или Chloros CLI и войдя в систему с помощью своих учетных данных. Это нужно сделать только один раз. В Linux (без графического интерфейса) используйте: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Основное использование

Обработайте папку всего несколькими строками:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Кроссплатформенные пути**: В примерах кода на этой странице используются пути в стиле Windows (например, `C:\\DroneImages\\Flight001`). В Linux используйте вместо этого пути в формате Linux (например, `/home/user/drone_images/flight001` или `~/drone_images/flight001`). SDK работает одинаково на обеих платформах.
{% endhint %}

### Полный контроль

Для расширенных рабочих процессов:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Руководство по установке

### Необходимые условия

Перед установкой SDK убедитесь, что у вас есть:

1. **Chloros установлен** — Windows: Установщик для настольных компьютеров ([скачать](download.md)); Linux: пакет `.deb` ([Установка](linux/linux-installation.md))
2. **Python 3.7+** установлено ([python.org](https://www.python.org))
3. **Действующая лицензия Chloros+** ([обновление](https://cloud.mapir.camera/pricing))

### Установка через pip

**Стандартная установка:**

```bash
pip install chloros-sdk
```

**С поддержкой отслеживания хода установки:**

```bash
pip install chloros-sdk[progress]
```

**Установка для разработчиков:**

```bash
pip install chloros-sdk[dev]
```

### Проверка установки

Проверьте, правильно ли установлен SDK:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Первоначальная настройка

### Активация лицензии

SDK использует ту же лицензию, что и Chloros, Chloros (браузер) и Chloros CLI. Активируйте один раз через графический интерфейс или CLI:**Windows:**Откройте**Chloros или Chloros (браузер)** и войдите в систему на вкладке «Пользователь» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> или воспользуйтесь CLI.**Linux:** Используйте CLI (графический интерфейс недоступен):

```bash
chloros-cli login user@example.com 'your_password'
```

Лицензия кэшируется локально и сохраняется после перезагрузки.

{% hint style="success" %}
**Однократная настройка**: после входа в систему через графический интерфейс или CLI, SDK автоматически использует лицензию из кэша. Дополнительная аутентификация не требуется!
{% endhint %}

{% hint style="info" %}
**Выход**: Пользователи SDK могут программно очистить кэшированные учетные данные с помощью метода `logout()`. См. [метод logout()](#logout) в справочнике API.
{% endhint %}

### Проверка подключения

Убедитесь, что SDK может подключиться к Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Справочник API

### Класс ChlorosLocal

Основной класс для локальной обработки изображений Chloros.

#### Конструктор

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Параметры:**

| Параметр                 | Тип | По умолчанию                   | Описание                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL локального бэкэнда Chloros          |
| `auto_start_backend`      | bool | `True`                    | Автоматически запускать бэкэнд при необходимости |
| `backend_exe`             | str  | `None` (автоопределение)      | Путь к исполняемому файлу бэкэнда            |
| `timeout`                 | int  | `30`                      | Таймаут запроса в секундах            |
| `backend_startup_timeout` | int  | `60`                      | Таймаут запуска бэкэнда (секунды) |

**Примеры:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Кроссплатформенное автоопределение**: SDK автоматически пробует правильный путь к бэкенду для вашей платформы:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (вручную)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Методы

#### `create_project(project_name, camera=None)`

Создать новый проект Chloros.

**Параметры:**

| Параметр      | Тип | Обязателен | Описание                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Да      | Название проекта                                     |
| `camera`       | str  | Нет       | Шаблон камеры (например, «Survey3N\_RGN», «Survey3W\_OCN») |

**Возвращает:** `dict` — Ответ на создание проекта**Пример:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Импорт изображений из папки.

**Параметры:**

| Параметр     | Тип     | Обязателен | Описание                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Да      | Путь к папке с изображениями         |
| `recursive`   | bool     | Нет       | Искать в подпапках (по умолчанию: False) |

**Возвращает:** `dict` — результаты импорта с количеством файлов**Пример:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Настройка параметров обработки.

**Параметры:**

| Параметр                 | Тип | По умолчанию                 | Описание                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | «Стандартный (быстрый, среднее качество)» | Метод дебайеризации            |
| `vignette_correction`     | bool | `True`                  | Включить коррекцию виньетирования      |
| `reflectance_calibration` | bool | `True`                  | Включить калибровку отражательной способности  |
| `indices`                 | список | `None`                  | Рассчитываемые индексы растительности |
| `export_format`           | стр.  | «TIFF (16-бит)»         | Формат вывода                   |
| `ppk`                     | bool | `False`                 | Включить поправки PPK          |
| `custom_settings`         | dict | `None`                  | Расширенные пользовательские настройки        |

**Форматы экспорта:**

* `"TIFF (16-bit)"` — рекомендуется для ГИС/фотограмметрии
* `"TIFF (32-bit, Percent)"` — научный анализ
* `"PNG (8-bit)"` — визуальный осмотр
* `"JPG (8-bit)"` — сжатый вывод

**Доступные индексы:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 и др.**Пример:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Обработать изображения проекта.

**Параметры:**

| Параметр           | Тип     | По умолчанию      | Описание                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Режим обработки: «parallel» или «serial»   |
| `wait`              | bool     | `True`       | Ожидание завершения                       |
| `progress_callback` | callable | `None`       | Функция обратного вызова прогресса (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Интервал опроса прогресса (секунды)   |

**Возвращает:** `dict` - Результаты обработки

{% hint style="warning" %}
**Параллельный режим**: Требуется лицензия Chloros+. Автоматически масштабируется в соответствии с количеством ядер вашего процессора (до 16 рабочих процессов).
{% endhint %}

**Пример:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Получить текущую конфигурацию проекта.

**Возвращает:** `dict` — текущую конфигурацию проекта**Пример:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Получить информацию о состоянии бэкэнда, включая ход обработки по каждому потоку.

**Возвращает:** `dict` — состояние бэкэнда со следующей структурой:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Пример:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Завершение работы бэкэнда (если он был запущен с помощью SDK).

**Пример:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Очистка кэшированных учетных данных из локальной системы.

**Описание:**

Программный выход из системы путем удаления кэшированных учетных данных. Это полезно для:
* Переключения между различными учетными записями Chloros+
* Очистки учетных данных в автоматизированных средах
* Целей безопасности (например, удаление учетных данных перед удалением программы)

**Возвращает:** `dict` — результат операции выхода из системы**Пример:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Требуется повторная аутентификация**: после вызова `logout()` необходимо повторно войти в систему с помощью Chloros, Chloros (браузер) или Chloros CLI перед использованием SDK.
{% endhint %}

***

### Удобные функции

#### `process_folder(folder_path, **options)`

Однострочная удобная функция для обработки папки.

**Параметры:**

| Параметр                 | Тип     | По умолчанию         | Описание                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Обязателен        | Путь к папке с изображениями     |
| `project_name`            | str      | Автоматически сгенерированный  | Название проекта                   |
| `camera`                  | str      | `None`          | Шаблон камеры                |
| `indices`                 | список     | `["NDVI"]`      | Индексы для расчета           |
| `vignette_correction`     | bool     | `True`          | Включить коррекцию виньетирования     |
| `reflectance_calibration` | bool     | `True`          | Включить калибровку отражательной способности |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Формат вывода                  |
| `mode`                    | str      | `"parallel"`    | Режим обработки                |
| `progress_callback`       | callable | `None`          | Обратный вызов прогресса              |

**Возвращает:** `dict` — результаты обработки**Пример:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Поддержка менеджеров контекста

SDK поддерживает менеджеры контекста для автоматической очистки:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Полные примеры

{% hint style="info" %}
**Пользователи Linux**: Во всех приведенных ниже примерах используются пути Windows. Замените пути `C:\\...` на ваши пути Linux (например, `/home/user/...` или `~/...`). Все функции SDK одинаковы на всех платформах.
{% endhint %}

### Пример 1: Базовая обработка

Обработка папки с настройками по умолчанию:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Пример 2: Пользовательский рабочий процесс

Полный контроль над конвейером обработки:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Пример 3: Пакетная обработка нескольких папок

Обработка нескольких наборов данных полетов:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Пример 4: Интеграция в исследовательский конвейер

Интеграция Chloros с анализом данных:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Пример 5: Настраиваемый мониторинг хода выполнения

Расширенное отслеживание хода выполнения с ведением журнала:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Пример 6: Обработка ошибок

Надежная обработка ошибок для использования в производственной среде:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Пример 7: Управление учетными записями и выход из системы

Управление учетными данными программным способом:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Пример 8: Инструмент командной строки

Создание настраиваемого инструмента CLI с помощью SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Использование:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Обработка исключений

SDK предоставляет специальные классы исключений для различных типов ошибок:

### Иерархия исключений

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Примеры исключений

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Расширенные темы

### Настройка пользовательского бэкэнда

Используйте пользовательское расположение или конфигурацию бэкэнда:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Неблокирующая обработка

Запустите обработку и продолжайте выполнение других задач:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Управление памятью

Для больших наборов данных выполняйте обработку пакетами:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Устранение неполадок

### Бэкэнд не запускается

**Проблема:** SDK не может запустить бэкэнд**Решения:**

1. Убедитесь, что Chloros установлен:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Проверьте брандмауэр (Windows) или доступность порта (Linux: `lsof -i :5000`)
3. Попробуйте указать путь к бэкенду вручную:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Лицензия не обнаружена**Проблема:** SDK выдает предупреждение об отсутствии лицензии**Решения:**

1. Откройте Chloros, Chloros (браузер) или Chloros CLI и войдите в систему.
2. Убедитесь, что лицензия кэширована:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Если возникают проблемы с учетными данными, очистите кэшированные учетные данные и войдите в систему заново:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Обратитесь в службу поддержки: info@mapir.camera

***

### Ошибки импорта**Проблема:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Решения:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Превышение времени ожидания обработки**Проблема:** Превышение времени ожидания обработки**Решения:**

1. Увеличьте время ожидания:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Обрабатывайте меньшие партии
3. Проверьте доступное место на диске
4. Контролируйте системные ресурсы

***

### Порт уже занят**Проблема:** Порт бэкэнда 5000 занят**Решения:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Или найдите и закройте конфликтующий процесс:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Советы по повышению производительности

### Оптимизация скорости обработки

1. **Используйте параллельный режим** (требуется Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Уменьшите разрешение вывода** (если это допустимо)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Отключите ненужные индексы**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Обрабатывайте на SSD** (а не на HDD)***

### Оптимизация памяти

Для больших наборов данных:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Фоновая обработка

Освободите Python для других задач:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Примеры интеграции

### Интеграция с Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Часто задаваемые вопросы

### В: Требуется ли для SDK подключение к Интернету?

**О:** Только для первоначальной активации лицензии. После входа в систему через Chloros, Chloros (браузер) или Chloros CLI лицензия кэшируется локально и работает в автономном режиме в течение 30 дней.***

### В: Можно ли использовать SDK на сервере без графического интерфейса?**О:** Да! SDK работает в режиме без графического интерфейса как на серверах Windows, так и на серверах Linux.**Linux (рекомендуется для работы без графического интерфейса):**
* Установка через пакет `.deb`
* Активация лицензии: `chloros-cli login user@example.com 'password'`

**Сервер Windows:**
* Сервер Windows 2016 или более поздней версии
* Установлен Chloros (однократно)
* Лицензия активирована через CLI или на любом компьютере

***

### В: В чем разница между Desktop, CLI и SDK?

| Функция         | Графический интерфейс Desktop | Командная строка CLI | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Интерфейс**   | Управление мышью | Командная строка | Python API  |
| **Наилучшее применение** | Визуальная работа | Скрипты        | Интеграция |
| **Автоматизация**  | Ограниченная     | Хорошая             | Отличная   |
| **Гибкость** | Базовая       | Хорошая             | Максимальная     |
| **Лицензия**     | Chloros+    | Chloros+         | Chloros+    |***

### В: Могу ли я распространять приложения, созданные с помощью SDK?**О:** Код SDK можно интегрировать в ваши приложения, но:

* Конечным пользователям необходимо установить Chloros
* Конечным пользователям необходимы действующие лицензии Chloros+
* Для коммерческого распространения требуется OEM-лицензия

По вопросам OEM обращайтесь в info@mapir.camera.

***

### В: Как обновить SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### В: Где сохраняются обработанные изображения?

По умолчанию — в папке Project Path :

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### В: Можно ли обрабатывать изображения из скриптов Python, запускаемых по расписанию?**О:** Да! Используйте планировщик вашей ОС со скриптами Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Запланируйте ежедневное выполнение через Планировщик заданий.**Linux:** Запланируйте через cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### В: Поддерживает ли SDK async/await?**О:** Текущая версия является синхронной. Для асинхронного поведения используйте `wait=False` или запускайте в отдельном потоке:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### В: Как переключаться между разными учетными записями Chloros+?**О:** Используйте метод `logout()` для очистки кэшированных учетных данных, затем повторно войдите в систему с новой учетной записью:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

После выхода из системы пройдите аутентификацию с новой учетной записью через графический интерфейс, браузер или CLI, прежде чем снова использовать SDK.

***

## Получение помощи

### Документация

* **Справочник API**: Эта страница

### Каналы поддержки

* **Электронная почта**: info@mapir.camera
* **Веб-сайт**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Цены**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Примеры кода

Все приведенные здесь примеры протестированы и готовы к использованию в производственной среде. Скопируйте и адаптируйте их для своего случая использования.

***

## Лицензия**Проприетарное программное обеспечение** — Copyright (c) 2025 MAPIR Inc.

Для работы SDK требуется действующая подписка Chloros+. Несанкционированное использование, распространение или модификация запрещены.
