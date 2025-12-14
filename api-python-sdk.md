# API : Python SDK

**Chloros Python SDK** обеспечивает программный доступ к движку обработки изображений Chloros, позволяя автоматизировать процессы, настраивать рабочие потоки и обеспечивать беспрепятственную интеграцию с вашими приложениями Python и исследовательскими конвейерами.

### Ключевые особенности

* 🐍 **Нативный Python** - Чистый, питонический API для обработки изображений
* 🔧 **Полный доступ к API** - Полный контроль над обработкой Chloros
* 🚀 **Автоматизация** - Создание настраиваемых рабочих процессов пакетной обработки
* 🔗 **Интеграция** - Встраивайте Chloros в существующие приложения Python
* 📊 **Готовность к исследованиям** - Идеально подходит для научных аналитических конвейеров
* ⚡ **Параллельная обработка** - Масштабируется до ваших ядер ЦП (Chloros+)

### Требования

| Требование          | Подробности                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros Desktop**  | Должно быть установлено локально                                           |
| **Лицензия**          | Chloros+ ([требуется платный план](https://cloud.mapir.camera/pricing)) |
| **Операционная система** | Windows 10/11 (64-разрядная)                                              |
| **Python**           | Python 3.7 или выше                                                |
| **Память**           | Минимум 8 ГБ ОЗУ (рекомендуется 16 ГБ)                                  |
| **Интернет**         | Требуется для активации лицензии                                     |

{% hint style=&quot;warning&quot; %}
**Требования к лицензии**: Python SDK требует платной подписки Chloros+ для доступа к API. Стандартные (бесплатные) тарифные планы не предоставляют доступ к API/SDK. Посетите [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), чтобы перейти на более дорогой тарифный план.
{% endhint %}

## Быстрый старт

### Установка

Установите через pip:

```bash
pip install chloros-sdk
```

{% hint style=&quot;info&quot; %}
**Первоначальная настройка**: Перед использованием SDK активируйте лицензию Chloros+, открыв Chloros, Chloros (браузер) или Chloros CLI и войдите в систему, используя свои учетные данные. Это нужно сделать только один раз.
{% endhint %}

### Основное использование

Обработайте папку всего несколькими строками:

```python
from chloros_sdk import process_folder

# One-line processing
results = process_folder("C:\\DroneImages\\Flight001")
```

### Полный контроль

Для расширенных рабочих процессов:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")

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

1. **Chloros Desktop** ([скачать](download.md))
2. Установлен **Python 3.7+** ([python.org](https://www.python.org))
3. **Активная лицензия Chloros+** ([обновление](https://cloud.mapir.camera/pricing))

### Установка через pip

**Стандартная установка:**

```bash
pip install chloros-sdk
```

**С поддержкой мониторинга прогресса:**

```bash
pip install chloros-sdk[progress]
```

**Установка для разработчиков:**

```bash
pip install chloros-sdk[dev]
```

### Проверка установки

Проверьте, что SDK установлен правильно:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Первоначальная настройка

### Активация лицензии

SDK использует ту же лицензию, что и Chloros, Chloros (браузер) и Chloros CLI. Активируйте один раз через графический интерфейс пользователя или CLI:

1. Откройте **Chloros или Chloros (браузер)** и войдите в систему на вкладке «Пользователь» <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> . Или откройте **CLI**.
2. Введите свои учетные данные Chloros+ и войдите в систему
3. Лицензия кэшируется локально (сохраняется после перезагрузки)

{% hint style=&quot;success&quot; %}
**Однократная настройка**: после входа в систему через графический интерфейс или CLI, SDK автоматически использует кэшированную лицензию. Дополнительная аутентификация не требуется!
{% endhint %}

### Проверка соединения

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
| `backend_exe`             | str  | `None` (автоматическое определение)      | Путь к исполняемому файлу бэкэнда            |
| `timeout`                 | int  | `30`                      | Таймаут запроса в секундах            |
| `backend_startup_timeout` | int  | `60`                      | Таймаут запуска бэкэнда (секунды) |

**Примеры:**

```python
# Default (auto-start backend)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom timeout
chloros = ChlorosLocal(timeout=60)
```

***

### Методы

#### `create_project(project_name, camera=None)`

Создать новый проект Chloros.

**Параметры:**

| Параметр      | Тип | Обязательный | Описание                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Да      | Название проекта                                     |
| `camera`       | str  | Нет       | Шаблон камеры (например, «Survey3N\_RGN», «Survey3W\_OCN») |

**Возвращает:** `dict` — ответ на создание проекта.

**Пример:**

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

| Параметр     | Тип     | Обязательный | Описание                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Да      | Путь к папке с изображениями         |
| `recursive`   | bool     | Нет       | Поиск в подпапках (по умолчанию: False) |

**Возвращает:** `dict` — результаты импорта с количеством файлов.

**Пример:**

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
| `debayer`                 | str  | «Высокое качество (быстрее)» | Метод дебайеризации                  |
| `vignette_correction`     | bool | `True`                  | Включить коррекцию виньетки      |
| `reflectance_calibration` | bool | `True`                  | Включить калибровку отражения  |
| `indices`                 | список | `None`                  | Рассчитываемые индексы растительности |
| `export_format`           | str  | &quot;TIFF (16-bit)&quot;         | Формат вывода                   |
| `ppk`                     | bool | `False`                 | Включить PPK-поправки          |
| `custom_settings`         | dict | `None`                  | Расширенные настройки        |

**Форматы экспорта:**

* `"TIFF (16-bit)"` — рекомендуется для ГИС/фотограмметрии
* `"TIFF (32-bit, Percent)"` — научный анализ
* `"PNG (8-bit)"` — визуальный осмотр
* `"JPG (8-bit)"` — сжатый вывод

**Доступные индексы:**

NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 и другие.

**Пример:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="High Quality (Faster)",
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
| `mode`              | str      | `"parallel"` | Режим обработки: «параллельный» или «последовательный»   |
| `wait`              | bool     | `True`       | Ожидать завершения                       |
| `progress_callback` | callable | `None`       | Функция обратного вызова прогресса (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Интервал опроса для прогресса (секунды)   |

**Возвращает:** `dict` — результаты обработки

{% hint style=&quot;warning&quot; %}
**Параллельный режим**: Требуется лицензия Chloros+. Автоматически масштабируется до ваших ядер ЦП (до 16 рабочих процессов).
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

**Возвращает:** `dict` — текущую конфигурацию проекта.

**Пример:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Получить информацию о состоянии бэкэнда.

**Возвращает:** `dict` — состояние бэкэнда

**Пример:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
```

***

#### `shutdown_backend()`

Завершение работы бэкэнда (если он был запущен с помощью SDK).

**Пример:**

```python
chloros.shutdown_backend()
```

***

### Удобные функции

#### `process_folder(folder_path, **options)`

Удобная однострочная функция для обработки папки.

**Параметры:**

| Параметр                 | Тип     | По умолчанию         | Описание                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Обязательный        | Путь к папке с изображениями     |
| `project_name`            | str      | Автоматически сгенерированный  | Название проекта                   |
| `camera`                  | str      | `None`          | Шаблон камеры                |
| `indices`                 | list     | `["NDVI"]`      | Индексы для расчета           |
| `vignette_correction`     | bool     | `True`          | Включить коррекцию виньетирования     |
| `reflectance_calibration` | bool     | `True`          | Включить калибровку отражения |
| `export_format`           | str      | &quot;TIFF (16-bit)&quot; | Формат вывода                  |
| `mode`                    | str      | `"parallel"`    | Режим обработки                |
| `progress_callback`       | callable | `None`          | Обратная связь по ходу выполнения              |

**Возвращаемые значения:** `dict` — результаты обработки

**Пример:**

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

## Поддержка контекстных менеджеров

SDK поддерживает контекстные менеджеры для автоматической очистки:

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
    debayer="High Quality (Faster)",
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

Обработка нескольких наборов данных о полетах:

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

### Пример 4: Интеграция исследовательского конвейера

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

### Пример 5: Настраиваемый мониторинг прогресса

Расширенное отслеживание прогресса с ведением журнала:

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

Надежная обработка ошибок для производственного использования:

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
        return False, f"Backend error: {e}. Ensure Chloros Desktop is installed."
    
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

### Пример 7: Инструмент командной строки

Создайте настраиваемый инструмент CLI с помощью SDK:

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
    
    args = parser.parse_args()
    
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
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI
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
    print("Backend failed to start. Ensure Chloros Desktop is installed.")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Расширенные темы

### Настройка пользовательского бэкэнда

Используйте настраиваемое расположение или конфигурацию бэкэнда:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Неблокирующая обработка

Начните обработку и продолжайте выполнять другие задачи:

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

**Проблема:** SDK не может запустить бэкэнд.

**Решения:**

1. Убедитесь, что Chloros Desktop установлен:

```python
import os
backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Убедитесь, что брандмауэр Windows не блокирует
3. Попробуйте вручную указать путь к бэкенду:

```python
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")
```

***

### Лицензия не обнаружена

**Проблема:** SDK выдает предупреждение об отсутствии лицензии.

**Решения:**

1. Откройте Chloros, Chloros (браузер) или Chloros CLI и войдите в систему.
2. Убедитесь, что лицензия кэширована:

```python
from pathlib import Path
import os

# Check cache location (Windows)
cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
print(f"Cache exists: {cache_path.exists()}")
```

3. Обратитесь в службу поддержки: info@mapir.camera

***

### Ошибки импорта

**Проблема:** `ModuleNotFoundError: No module named 'chloros_sdk'`

**Решения:**

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

### Время ожидания обработки

**Проблема:** Истечение времени обработки

**Решения:**

1. Увеличьте время ожидания:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Обрабатывайте меньшие партии
3. Проверьте доступное место на диске
4. Контролируйте системные ресурсы

***

### Порт уже используется

**Проблема:** Порт бэкэнда 5000 занят

**Решения:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Или найдите и закройте конфликтующий процесс:

```powershell
# PowerShell
Get-NetTCPConnection -LocalPort 5000
```

***

## Советы по повышению производительности

### Оптимизация скорости обработки

1. **Используйте параллельный режим** (требуется Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Уменьшите разрешение вывода** (если это приемлемо)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Отключите ненужные индексы**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Обработка на SSD** (не на HDD)

***

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

**О:** Только для первоначальной активации лицензии. После входа в систему через Chloros, Chloros (браузер) или Chloros CLI лицензия кэшируется локально и работает в автономном режиме в течение 30 дней.

***

### В: Можно ли использовать SDK на сервере без графического интерфейса?

**О:** Да! Требования:

* Windows Server 2016 или более поздней версии
* Chloros установлен (однократно)
* Лицензия активирована на любом компьютере (кэшированная лицензия скопирована на сервер)

***

### В: В чем разница между Desktop, CLI и SDK?

| Функция         | Desktop GUI | CLI Command Line | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Интерфейс**   | Точечный клик | Командная строка          | Python API  |
| **Лучше всего подходит для**    | Визуальной работы | Скриптинга        | Интеграции |
| **Автоматизация**  | Ограниченная     | Хорошая             | Отличная   |
| **Гибкость** | Базовая       | Хорошая             | Максимальная     |
| **Лицензия**     | Chloros+    | Chloros+         | Chloros+    |

***

### В: Могу ли я распространять приложения, созданные с помощью SDK?

**О:** Код SDK может быть интегрирован в ваши приложения, но:

* Конечным пользователям необходимо установить Chloros.
* Конечным пользователям необходимы активные лицензии Chloros+.
* Для коммерческого распространения требуется лицензия OEM.

По вопросам OEM обращайтесь в info@mapir.camera.

***

### В: Как обновить SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### В: Где сохраняются обработанные изображения?

По умолчанию в пути проекта:

```
Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### В: Можно ли обрабатывать изображения из скриптов Python, запущенных по расписанию?

**О:** Да! Используйте Windows Task Scheduler со скриптами Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("C:\\Flights\\Today")
```

Запланируйте ежедневное выполнение с помощью Task Scheduler.

***

### В: Поддерживает ли SDK асинхронные/ожидающие операции?

**О:** Текущая версия является синхронной. Для асинхронного поведения используйте `wait=False` или запустите в отдельном потоке:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

## Получение помощи

### Документация

* **Справочник API**: эта страница

### Каналы поддержки

* **Электронная почта**: info@mapir.camera
* **Веб-сайт**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Цены**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Примеры кода

Все приведенные здесь примеры протестированы и готовы к использованию. Скопируйте и адаптируйте их для своего случая использования.

***

## Лицензия

**Проприетарное программное обеспечение** — Copyright (c) 2025 MAPIR Inc.

SDK требует активной подписки Chloros+. Несанкционированное использование, распространение или модификация запрещены.
