# Установка Linux

Chloros распространяется для Linux в виде пакетов `.deb`, которые устанавливают CLI и бэкэнд. Python SDK устанавливается отдельно через pip.

***

## Linux amd64 (x86_64)

### Системные требования

| Требование | Минимальное | Рекомендуемое |
| --- | --- | --- |
| **Дистрибутив** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Процессор** | x86_64 (Intel/AMD) | Intel Core i7 или лучше |
| **Память (RAM)** | 8 ГБ | 16 ГБ или больше |
| **Видеокарта** | Не требуется (обработка на процессоре) | Видеокарта NVIDIA с 4 ГБ+ видеопамяти |
| **Хранение** | 2 ГБ свободного места | SSD с 10 ГБ+ свободного места |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

### Установка

Загрузите пакет `.deb` и установите:

```bash
sudo dpkg -i chloros-amd64.deb
```

Проверьте установку:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Системные требования

| Требование | Минимальное | Рекомендуемое |
| --- | --- | --- |
| **Платформа** | NVIDIA Jetson с JetPack 6 | Jetson Orin NX 16 ГБ или AGX Orin |
| **JetPack** | JetPack 6.x | Последнюю версию JetPack 6 |
| **Память (RAM)** | 8 ГБ (общая для GPU/CPU) | 16 ГБ+ (общая) |
| **Хранение** | 2 ГБ свободного места | SSD NVMe с 10 ГБ+ свободного места |
| **Python** | Python 3.7+ (для SDK) | Python 3.10+ |

### Установка

Загрузите пакет JetPack 6 `.deb` и установите:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Проверьте установку:

```bash
chloros-cli --version
```

Подробную информацию о настройке Jetson, включая управление тепловым режимом и развертывание в полевых условиях, см. в [Руководстве по NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Python SDK Установка (все Linux)

Python SDK устанавливается отдельно с помощью pip и работает как на amd64, так и на arm64:

```bash
pip install chloros-sdk
```

Чтобы включить дополнительную поддержку потоковой передачи прогресса:

```bash
pip install chloros-sdk[progress]
```

Проверьте SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
Пакет `.deb` устанавливает Chloros, CLI и бэкэнд. Python SDK — это отдельный пакет pip, который взаимодействует с бэкэндом через локальный HTTP API.
{% endhint %}

***

## Конфигурационные каталоги

Chloros на Linux соответствует [Спецификации базового каталога XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Назначение | Linux Путь | Windows Эквивалент |
| --- | --- | --- |
| **Конфигурация** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Данные / Проекты** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Кэш / Учетные данные** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Расположение исполняемых файлов бэкэнда

Пакет `.deb` устанавливает бэкэнд в стандартное место. Пакеты CLI и SDK автоматически определяют путь к бэкенду:

| Метод установки | Путь к бэкенду |
| --- | --- |
| Пакет `.deb` | `/usr/lib/chloros/chloros-backend` |
| Ручной / пользовательский | `/opt/mapir/chloros/backend/chloros-backend` |

Вы можете переопределить путь к бэкенду с помощью флага `--backend-exe` CLI или параметра конструктора `backend_exe` SDK.

***

## Первоначальная настройка

### 1. Активируйте лицензию

Для доступа к CLI и SDK требуется лицензия Chloros+:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Проверьте статус лицензии

```bash
chloros-cli status
```

### 3. Обработайте первый набор данных

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Запустите диагностику системы

Убедитесь, что ваша система настроена правильно:

```bash
chloros-cli selftest
```

Это запускает 7 диагностических проверок, включая проверку версии, запуск бэкэнда, API подключение и доступность CUDA/GPU.

***

## Примеры скриптов Bash

### Обработка нескольких наборов данных

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Обработка с пользовательскими настройками

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Автоматическая обработка с помощью Cron

Добавьте в crontab (`crontab -e`) для автоматической обработки новых наборов данных:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Пример Python SDK

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

Если `chloros-cli` не найден после установки пакета `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Отказ в доступе

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Не удалось запустить бэкэнд

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA не обнаружена

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Отсутствуют общие библиотеки

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Обновление Chloros на Linux

Используйте встроенную команду обновления, чтобы проверить наличие обновлений и установить их:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Следующие шаги

* [Руководство по NVIDIA Jetson](nvidia-jetson-guide.md) — Оптимизация и развертывание для Jetson
* [CLI : Командная строка](../CLI.md) — Полный справочник команд CLI
* [API : Python SDK](../api-python-sdk.md) — Полное справочное руководство по SDK
* [Динамическая адаптация вычислений](../processing-architecture/dynamic-compute-adaptation.md) — Как Chloros адаптируется к вашему оборудованию
