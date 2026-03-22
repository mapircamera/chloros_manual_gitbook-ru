# CLI : Командная строка

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>**Chloros CLI** предоставляет мощный доступ из командной строки к движку обработки изображений Chloros, позволяя автоматизировать, создавать скрипты и работать в режиме без интерфейса для ваших рабочих процессов обработки изображений.

### Ключевые особенности

* 🚀 **Автоматизация** — пакетная обработка нескольких наборов данных с помощью скриптов
* 🔗 **Интеграция** — встраивание в существующие рабочие процессы и конвейеры
* 💻 **Работа в режиме без интерфейса** — запуск без графического интерфейса
* 🌍 **Многоязычность** — поддержка 38 языков
* ⚡ **Параллельная обработка** — [Динамическая адаптация вычислений](processing-architecture/dynamic-compute-adaptation.md) автоматически оптимизирует работу под ваше оборудование

### Требования

| Требование          | Подробности                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Операционная система** | Windows 10/11 (64-битная), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Лицензия**          | Chloros+ ([требуется платный тарифный план](https://cloud.mapir.camera/pricing)) |
| **Память**           | Минимум 8 ГБ ОЗУ (рекомендуется 16 ГБ)                                  |
| **Интернет**         | Требуется для активации лицензии                                     |
| **Место на диске**       | Зависит от размера проекта                                              |

{% hint style="warning" %}
**Требования к лицензии**: Для CLI требуется платная подписка Chloros+. Стандартные (бесплатные) тарифные планы не предоставляют доступ к CLI. Перейдите на страницу [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing), чтобы перейти на более дорогой тарифный план.
{% endhint %}

## Быстрый старт

### Установка

#### Windows

CLI автоматически включается в установщик Chloros:

1. Загрузите и запустите **Chloros Installer.exe**

2. Завершите работу мастера установки
3. CLI установлен в: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
Установщик автоматически добавляет `chloros-cli` в системный PATH. Перезапустите терминал после установки.
{% endhint %}

#### Linux

Установите пакет `.deb` для вашей архитектуры:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Подробную информацию о настройке Linux см. в разделе [Установка Linux](linux/linux-installation.md).

### Первоначальная настройка

Перед использованием CLI активируйте лицензию Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Основное использование

Обработка папки с настройками по умолчанию:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Справочник команд

### Общий синтаксис

```
chloros-cli [global-options] <command> [command-options]
```

***

## Команды

### `process` — Обработка изображений

Обработка изображений в папке с калибровкой.

**Синтаксис:**

```bash
chloros-cli process <input-folder> [options]
```

**Примеры:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Параметры команды обработки

| Параметр                | Тип    | По умолчанию        | Описание                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Путь    | _Обязательно_     | Папка, содержащая мультиспектральные изображения в формате RAW/JPG                                         |
| `-o, --output`        | Путь    | То же, что и входные данные  | Папка для вывода обработанных изображений                                                     |
| `-n, --project-name`  | Строка  | Автоматически сгенерировано | Пользовательское имя проекта                                                                    |
| `--vignette`          | Флаг    | Включено        | Включить коррекцию виньетирования                                                             |
| `--no-vignette`       | Флаг    | -              | Отключить коррекцию виньетирования                                                            |
| `--reflectance`       | Флаг    | Включено        | Включить калибровку отражательной способности                                                         |
| `--no-reflectance`    | Флаг    | -              | Отключить калибровку отражательной способности                                                        |
| `--ppk`               | Флаг    | Отключено       | Применить поправки PPK на основе данных датчика освещенности .daq                                      |
| `--format`            | Выбор  | TIFF (16-бит)  | Формат вывода: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Целое число | Авто           | Минимальный размер цели в пикселях для обнаружения калибровочной панели                          |
| `--target-clustering` | Целое число | Авто           | Пороговое значение кластеризации целей (0–100)                                                    |
| `--debayer`           | Выбор  | `standard`     | Метод дебайеризации: `standard` или `texture-aware` (только для Chloros+)                          |
| `--target`, `--targets` | Флаг  | Отключено       | Искать калибровочные цели только в подпапке «target» или «targets» (ускоряет обработку) |
| `--indices`           | Список    | Нет           | Индексы растительности для расчета (например, `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | Строка  | Нет           | Зафиксировать экспозицию для модели камеры (контакт 1)                                                 |
| `--exposure-pin-2`    | Строка  | Нет           | Фиксация экспозиции для модели камеры (контакт 2)                                                 |
| `--recal-interval`    | Целое число | Автоматически           | Интервал повторной калибровки в секундах                                                      |
| `--timezone-offset`   | Целое число | 0              | Смещение часового пояса в часах                                                               |

***

### `login` — Аутентификация учетной записи

Войдите в систему, используя свои учетные данные Chloros+, чтобы включить обработку CLI.

**Синтаксис:**

```bash
chloros-cli login <email> <password>
```

**Пример:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Специальные символы**: Используйте одинарные кавычки вокруг паролей, содержащих символы, такие как `$`, `!`, или пробелы.
{% endhint %}

**Вывод:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` — Очистить учетные данные

Очистить сохраненные учетные данные и выйти из учетной записи.

**Синтаксис:**

```bash
chloros-cli logout
```

**Пример:**

```bash
chloros-cli logout
```

**Вывод:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Пользователи SDK**: Python SDK также предоставляет программный метод `logout()` для очистки учетных данных в скриптах Python. Подробности см. в [документации по Python SDK](api-python-sdk.md#logout).
{% endhint %}

***

### `status` — Проверка статуса лицензии

Отображение текущего статуса лицензии и аутентификации.

**Синтаксис:**

```bash
chloros-cli status
```

**Пример:**

```bash
chloros-cli status
```

**Вывод:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` — Проверка хода экспорта

Мониторинг хода экспорта потока 4 во время или после обработки.

**Синтаксис:**

```bash
chloros-cli export-status
```

**Пример:**

```bash
chloros-cli export-status
```

**Сценарий использования:** Вызовите эту команду во время выполнения обработки, чтобы проверить ход экспорта.***

### `language` — Управление языком интерфейса

Просмотр или изменение языка интерфейса CLI.

**Синтаксис:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Примеры:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Поддерживаемые языки (всего 38)

| Код    | Язык              | Название на языке оригинала      |
| ------- | --------------------- | ---------------- |
| `en`    | Английский               | English          |
| `es`    | Испанский               | Español          |
| `pt`    | Португальский            | Português        |
| `fr`    | Французский                | Français         |
| `de`    | Немецкий                | Deutsch          |
| `it`    | Итальянский               | Italiano         |
| `ja`    | Японский              | 日本語              |
| `ko`    | Корейский                | 한국어              |
| `zh`    | Китайский (упрощённый)  | 简体中文             |
| `zh-TW` | Китайский (традиционный) | 繁體中文             |
| `ru`    | Русский               | Русский          |
| `nl`    | Нидерландский                | Nederlands       |
| `ar`    | Арабский                | العربية          |
| `pl`    | Польский                | Polski           |
| `tr`    | Турецкий               | Türkçe           |
| `hi`    | Хинди                 | हिंदी            |
| `id`    | Индонезийский            | Bahasa Indonesia |
| `vi`    | Вьетнамский            | Tiếng Việt       |
| `th`    | Тайский                  | ไทย              |
| `sv`    | Шведский               | Svenska          |
| `da`    | Датский                | Dansk            |
| `no`    | Норвежский             | Norsk            |
| `fi`    | Финский               | Suomi            |
| `el`    | Греческий                 | Ελληνικά         |
| `cs`    | Чешский                | Čeština          |
| `hu`    | Венгерский             | Magyar           |
| `ro`    | Румынский              | Română           |
| `uk`    | Украинский             | Українська       |
| `pt-BR` | Бразильский португальский  | Português Brasileiro |
| `zh-HK` | Кантонский             | 粵語             |
| `ms`    | Малайский                 | Bahasa Melayu    |
| `sk`    | Словацкий                | Slovenčina       |
| `bg`    | Болгарский             | Български        |
| `hr`    | Хорватский              | Hrvatski         |
| `lt`    | Литовский            | Lietuvių         |
| `lv`    | Латвийский               | Latviešu         |
| `et`    | Эстонский              | Eesti            |
| `sl`    | Словенский             | Slovenščina      |

{% hint style="success" %}
**Автоматическое сохранение**: Ваши языковые настройки сохраняются в `~/.chloros/cli_language.json` и сохраняются во всех сессиях.
{% endhint %}

***

### `set-project-folder` — Установить папку проекта по умолчанию

Изменить расположение папки проекта по умолчанию (общая с GUI в Windows).

**Синтаксис:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Примеры:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` — Показать папку проекта

Отобразить текущее расположение папки проекта по умолчанию.

**Синтаксис:**

```bash
chloros-cli get-project-folder
```

**Пример:**

```bash
chloros-cli get-project-folder
```

**Результат:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` — Сброс к значениям по умолчанию

Сброс папки проекта к расположению по умолчанию.

**Синтаксис:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` — Запуск диагностики системы

Запускает 7 диагностических проверок для проверки конфигурации системы.

**Синтаксис:**

```bash
chloros-cli selftest
```

**Выполняемые диагностические проверки:**

1. Проверка версии
2. Доступность порта (5000)
3. Запуск бэкэнда
4. Тест подключения API
5. Информация о системе и обнаружение графического процессора
6. Проверка моделей шумоподавителя
7. Проверка доступности CUDA

{% hint style="info" %}
**Полезно для устранения неполадок**: Запустите `selftest` после установки, чтобы убедиться, что ваша система настроена правильно, особенно на Linux/Jetson, где может потребоваться проверка настроек GPU и CUDA.
{% endhint %}

***

### `update` — Проверка наличия обновлений (только для Linux)

Проверка наличия и установка обновлений CLI на системах Linux.

**Синтаксис:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Параметр    | Описание                        |
| --------- | ---------------------------------- |
| `--check` | Только проверять наличие обновлений, не устанавливать |

{% hint style="info" %}
Эта команда доступна только в Linux. В Windows обновления предоставляются через установщик.
{% endhint %}

***

## Глобальные параметры

Эти параметры применимы ко всем командам:

| Параметр            | Тип    | По умолчанию       | Описание                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Путь    | Автоопределяется | Путь к исполняемому файлу бэкэнда                       |
| `--port`          | Целое число | 5000          | Номер порта бэкэнда API                          |
| `--restart`       | Флаг    | -             | Принудительный перезапуск бэкэнда (убивает существующие процессы) |
| `--version`       | Флаг    | -             | Показать информацию о версии и завершить работу                |
| `--help`          | Флаг    | -             | Показать справку и завершить работу                   |

{% hint style="info" %}
**Автоматическое определение бэкэнда**: Путь `--backend-exe` определяется автоматически для каждой платформы:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (ручная установка)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Пример с глобальными параметрами:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Руководство по настройкам обработки

### Параллельная обработка и динамическая адаптация вычислений

Chloros 1.1.0 включает [Динамическую адаптацию вычислений](processing-architecture/dynamic-compute-adaptation.md) — движок обработки **автоматически определяет ваше оборудование** и выбирает оптимальную стратегию:

| Платформа | Стратегия | Рабочие процессы | Конвейер | Примечания |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 ГБ** | `GPU_SINGLE` | 1 | `tiled_gpu` | Эффективное использование памяти, последовательная обработка |
| **Jetson Orin NX 16 ГБ** | `GPU_PARALLEL` | 3 | `fused_gpu` | Параллельная обработка на GPU |
| **Настольный компьютер с 8 ГБ GPU** | `GPU_SINGLE` | 3 | `tiled_gpu` | Хорошая производительность настольного компьютера |
| **Настольный компьютер с графическим процессором 12 ГБ+** | `GPU_PARALLEL` | 3–4 | `fused_gpu` | Оптимальная производительность настольного компьютера |
| **Система только с ЦП** | `CPU_PARALLEL` | ядер — 1 | `cpu_fallback` | Видеокарта не требуется |

{% hint style="success" %}
**Ручная настройка не требуется!** Chloros автоматически определяет ваш процессор, графический процессор, оперативную память и (на Jetson) датчики температуры, а затем автоматически настраивает оптимальный конвейер обработки.
{% endhint %}

### Методы дебайеризации

| Метод | Флаг CLI | Качество | Скорость | Лицензия |
| --- | --- | --- | --- | --- |
| **Стандартный (быстрый, среднее качество)** | `--debayer standard` | Хорошее | Быстрый | Бесплатно / Chloros+ |
| **С учетом текстуры (медленно, высочайшее качество)** | `--debayer texture-aware` | Высочайшее | Медленно | Только Chloros+ |

Метод дебайеризации по умолчанию — **Стандартный**. Метод**С учетом текстуры** использует модель шумоподавления на основе ИИ/машинного обучения для получения результатов высочайшего качества, но требует лицензии Chloros+ и графического процессора NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Коррекция виньетирования

**Что делает:** Корректирует падение освещенности по краям изображения (темные углы, характерные для камерных снимков).

* **Включено по умолчанию** — большинству пользователей следует оставить эту функцию включенной
* Используйте `--no-vignette` для отключения

{% hint style="success" %}
**Рекомендация**: Всегда включайте коррекцию виньетирования, чтобы обеспечить равномерную яркость по всему кадру.
{% endhint %}

### Калибровка отражательной способности

Преобразует исходные значения датчика в стандартизированные процентные значения отражательной способности с помощью калибровочных панелей.

* **Включено по умолчанию** — Необходимо для анализа растительности
* Требует наличия калибровочных панелей на изображениях
* Используйте `--no-reflectance` для отключения

{% hint style="info" %}
**Требования**: Убедитесь, что калибровочные панели правильно экспонированы и видны на ваших изображениях для точного преобразования отражательной способности.
{% endhint %}

### Коррекции PPK

**Что это делает:** Применяет постобработанные кинематические поправки с использованием данных журнала DAQ-A-SD для повышения точности GPS.

* **Отключено по умолчанию**
* Используйте `--ppk` для включения
* Требуются файлы .daq в папке проекта от датчика освещенности DAQ-A-SD MAPIR.

### Форматы вывода

<table><thead><tr><th width="197">Формат</th><th width="130.20001220703125">Разрядность</th><th width="116.5999755859375">Размер файла</th><th>Лучше всего подходит для</th></tr></thead><tbody><tr><td><strong>TIFF (16-бит)</strong> ⭐</td><td>16-битное целое число</td><td>Большой</td><td>Анализ ГИС, фотограмметрия (рекомендуется)</td></tr><tr><td><strong>TIFF (32-разрядное, проценты)</strong></td><td>32-разрядное число с плавающей запятой</td><td>Очень большие</td><td>Научный анализ, исследования</td></tr><tr><td><strong>PNG (8-бит)</strong></td><td>8-битное целое число</td><td>Средний</td><td>Визуальный осмотр, публикация в Интернете</td></tr><tr><td><strong>JPG (8-бит)</strong></td><td>8-разрядное целое число</td><td>Малый</td><td>Быстрый просмотр, сжатый вывод</td></tr></tbody></table>***

## Автоматизация и скрипты

### Пакетная обработка PowerShell (Windows)

Автоматическая обработка нескольких папок с наборами данных в Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Пакетный скрипт Windows (Windows)

Простой цикл для пакетной обработки в Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Пакетная обработка в Bash (Linux)

Обработка нескольких папок с наборами данных на Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Скрипт автоматизации Python (кроссплатформенный)

Расширенная автоматизация с обработкой ошибок (работает на Windows и Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Рабочий процесс обработки

### Стандартный рабочий процесс

1. **Входные данные**: папка, содержащая пары изображений в форматах RAW/JPG
2. **Обнаружение**: CLI автоматически сканирует файлы изображений, поддерживаемые программой
3. **Обработка**: параллельный режим масштабируется в зависимости от количества ядер вашего процессора (Chloros+)
4. **Вывод**: Создает подпапки по моделям камер с обработанными изображениями

### Пример структуры вывода

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Оценка времени обработки

Типичное время обработки 100 изображений (по 12 Мп каждое):

| Платформа | Режим | Ориентировочное время | Примечания |
| --- | --- | --- | --- |
| **Настольный ПК с GPU 12 ГБ+** | `GPU_PARALLEL` | 5–10 мин | Самый быстрый вариант |
| **Настольный компьютер с графическим процессором 8 ГБ** | `GPU_SINGLE` | 10–15 мин | Хорошая производительность |
| **Jetson Orin NX 16 ГБ** | `GPU_PARALLEL` | 15–25 мин | Пограничные вычисления |
| **Jetson Nano 8 ГБ** | `GPU_SINGLE` | 30–60 мин | Ограниченный объем памяти |
| **Только ЦП** | `CPU_PARALLEL` | 20–40 мин | ГП не требуется |

{% hint style="info" %}
**Совет по производительности**: Время обработки зависит от количества изображений, разрешения, метода дебайеризации и аппаратного обеспечения. Дебайеризация с учетом текстур занимает значительно больше времени, чем стандартная. Подробности см. в разделе [Динамическая адаптация вычислений](processing-architecture/dynamic-compute-adaptation.md).
{% endhint %}

***

## Устранение неполадок

### CLI не найден

**Windows Ошибка:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Решения:**

1. Проверьте место установки:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Используйте полный путь, если он не указан в PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Добавьте в PATH вручную:
   * Откройте «Свойства системы» → «Переменные среды»
   * Отредактируйте переменную PATH
   * Добавьте: `C:\Program Files\Chloros\resources\cli`
   * Перезапустите терминал

**Linux Ошибка:**

```
chloros-cli: command not found
```

**Linux Решения:**

1. Проверьте установку:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Перезагрузите оболочку:

```bash
source ~/.bashrc
```

3. Проверьте права доступа:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Не удалось запустить бэкэнд**Ошибка:**

```

Backend failed to start within 30 seconds
```

**Решения:**

1. Проверьте, не запущен ли бэкэнд (сначала закройте его)
2. Убедитесь, что брандмауэр не блокирует (Windows) или проверьте доступность порта (Linux: `lsof -i :5000`)
3. Попробуйте другой порт:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Принудительно перезапустите бэкэнд:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. В Linux проверьте наличие исполняемого файла бэкэнда:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Проблемы с лицензией / аутентификацией**Ошибка:**

```

Chloros+ license required for CLI access
```

**Решения:**

1. Убедитесь, что у вас есть действующая подписка Chloros+
2. Войдите в систему, используя свои учетные данные:

```bash
chloros-cli login user@example.com 'password'
```

3. Проверьте статус лицензии:

```bash
chloros-cli status
```

4. Обратитесь в службу поддержки: info@mapir.camera

***

### Изображения не найдены**Ошибка:**

```

No images found in the specified folder
```

**Решения:**

1. Убедитесь, что папка содержит поддерживаемые форматы (.RAW, .TIF, .JPG)
2. Проверьте правильность пути к папке (используйте кавычки для путей, содержащих пробелы)
3. Убедитесь, что у вас есть права на чтение для этой папки
4. Проверьте, верны ли расширения файлов

***

### Остановка или зависание обработки**Решения:**

1. Проверьте доступное место на диске (убедитесь, что его достаточно для вывода)
2. Закройте другие приложения, чтобы освободить память
3. Уменьшите количество изображений (обрабатывайте партиями)

***

### Порт уже используется**Ошибка:**

```

Port 5000 is already in use
```

**Решения:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Часто задаваемые вопросы

### В: Нужна ли мне лицензия для CLI?

**О:**Да! Для CLI требуется платная**лицензия Chloros+**.

* ❌ Стандартный (бесплатный) тариф: CLI отключен
* ✅ Тарифы Chloros+ (платные): CLI полностью включен

Подпишитесь по адресу: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### В: Можно ли использовать CLI на сервере без графического интерфейса?**О:** Да! CLI работает полностью в режиме без интерфейса. Это основной сценарий использования на Linux.**Сервер Windows:**
* Сервер Windows 2016 или более поздней версии
* Установлен Visual C++ Redistributable

**Сервер Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) или JetPack 6 (arm64)
* Установка через пакет `.deb`

**Обе платформы:**
* Минимум 8 ГБ ОЗУ (рекомендуется 16 ГБ)
* Однократная активация лицензии: `chloros-cli login user@example.com 'password'`

***

### В: Где сохраняются обработанные изображения?**О:**По умолчанию обработанные изображения сохраняются в**той же папке, что и исходные**, в подпапках по моделям камер (например, `Survey3N_RGN/`).

Используйте опцию `-o`, чтобы указать другую папку для вывода:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### В: Можно ли обрабатывать несколько папок одновременно?**A:** Не напрямую одной командой, но вы можете использовать скрипты для последовательной обработки папок. См. раздел [Автоматизация и скрипты](CLI.md#automation--scripting).***

### Q: Как сохранить вывод CLI в файл журнала?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### В: Что произойдет, если я нажму Ctrl+C во время обработки?**A:** CLI:

1. Плавно остановит обработку
2. Закроет бэкэнд
3. Выйдет с кодом 130

Частично обработанные изображения могут остаться в папке вывода.

***

### В: Можно ли автоматизировать обработку CLI?**О:** Конечно! CLI разработан для автоматизации. См. [Автоматизация и скрипты](CLI.md#automation--scripting) для PowerShell (Windows), Batch (Windows), Bash (Linux) и Python (кроссплатформенные).***

### В: Как проверить версию CLI?**О:**

```bash
chloros-cli --version
```

**Результат:**

```

Chloros CLI 1.1.0
```

***

## Получение справки

### Справка по командной строке

Просмотр справочной информации непосредственно в CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Каналы поддержки

* **Электронная почта**: info@mapir.camera
* **Веб-сайт**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Цены**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Полные примеры

### Пример 1: Базовая обработка

Обработка с настройками по умолчанию (виньетирование, отражательная способность):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Пример 2: Высококачественный научный результат

32-разрядная плавающая запятая TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Пример 3: Быстрая обработка предварительного просмотра

8-битный PNG без калибровки для быстрого просмотра:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Пример 4: Обработка с PPK-коррекцией

Применение PPK-коррекций с использованием коэффициента отражения:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Пример 5: Пользовательское местоположение вывода

Обработка в другое местоположение с определенным форматом:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Пример 6: Процесс аутентификации

Полный процесс аутентификации (одинаковый на всех платформах):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Пример 7: Использование нескольких языков

Изменение языка интерфейса (одинаковое на всех платформах):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
