# Руководство по NVIDIA Jetson

Chloros на платформе NVIDIA Jetson обеспечивает обработку мультиспектральных изображений на периферии — в полевых условиях, на БПЛА и в удаленных установках. Chloros автоматически определяет модель Jetson и оптимизирует стратегию обработки с учетом особенностей вашего оборудования.

***

## Поддерживаемые модели Jetson

| Модель                | ОЗУ            | Стратегия обработки                                   | Рекомендуемое использование                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32–64 ГБ общего | `GPU_PARALLEL` (4 рабочих процесса)                            | Максимальная производительность, большие наборы данных                      |
| **Jetson Orin NX**   | 8–16 ГБ общей памяти  | `GPU_PARALLEL` (3 рабочих процесса, 16 ГБ) / `GPU_SINGLE` (8 ГБ) | Основная рекомендация для развертывания в воздухе и в полевых условиях |
| **Jetson Orin Nano** | 8 ГБ общей памяти     | `GPU_SINGLE` (1 рабочий узел)                               | Пограничные вычисления начального уровня                                 |
| **Jetson Nano**      | 4–8 ГБ общей памяти   | `GPU_SINGLE` (1 рабочий процесс)                               | Начальный уровень, ограниченная память                          |

{% hint style="info" %}
**Устаревшие модели Jetson** (TX2, TX1, Xavier NX) могут не поддерживаться. Производительность будет зависеть от доступной памяти GPU и возможностей CUDA.
{% endhint %}

***

## Требования

* **JetPack 6.x** (рекомендуется последняя версия)
* **NVIDIA CUDA** (входит в состав JetPack)
* **Лицензия Chloros+** (требуется для доступа к CLI/SDK)

## Установка

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Общие сведения об установке Linux см. в разделе [Установка Linux](linux-installation.md).

***

## Динамическая адаптация вычислений на Jetson

Chloros автоматически определяет вашу модель Jetson и выбирает оптимальную стратегию обработки. **Ручная настройка не требуется.**

### Как это работает

При запуске Chloros анализирует вашу систему:

1. **Определяет модель Jetson** с помощью `/proc/device-tree/model`
2. **Считывает доступную память GPU/общую память**

3.**Выбирает стратегию обработки** (`GPU_PARALLEL`, `GPU_SINGLE` или `CPU_PARALLEL`)
4. **Автоматически устанавливает количество рабочих процессов, тип конвейера и распределение памяти**

### Поведение для каждой модели

| Модель Jetson                | Стратегия       | Рабочие процессы | Конвейер                       | Параллелизм |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 ГБ**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (эффективное использование памяти) | Последовательное  |
| **Jetson Orin Nano 8 ГБ**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Последовательная  |
| **Jetson Orin NX 8 ГБ**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Сериализованный  |
| **Jetson Orin NX 16 ГБ**     | `GPU_PARALLEL` | 3       | `fused_gpu` (полный путь GPU)    | Параллельный  |
| **Jetson AGX Orin 32–64 ГБ** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Параллельный  |

{% hint style="success" %}
**Jetson Orin NX 16 ГБ** — идеальный вариант для развертывания на периферии: он поддерживает стратегию `GPU_PARALLEL` с 3 параллельными рабочими процессами, обеспечивая настоящую параллельную обработку на GPU в компактном форм-факторе.
{% endhint %}

Ключевое различие между платформами заключается в **памяти**. Jetson Nano с 8 ГБ общей памяти должен обрабатывать изображения по одному, используя эффективный с точки зрения памяти подход с разбиением на фрагменты, в то время как Orin NX с 16 ГБ может одновременно обрабатывать 3 изображения на GPU, используя конвергированный конвейер с более высокой пропускной способностью.

Полную справку по адаптации вычислений см. в разделе [Динамическая адаптация вычислений](../processing-architecture/dynamic-compute-adaptation.md).

***

## Управление тепловым режимом

Устройства Jetson имеют ограниченный тепловой запас, особенно при размещении в закрытых помещениях или на борту воздушных судов. Chloros включает автоматический мониторинг температуры и регулирование мощности:

| Температура         | Действие                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Нормальная работа — полная скорость обработки          |
| **70 °C** (Предупреждение)  | Автоматическое уменьшение размера пакета                   |
| **80 °C** (Критическое состояние) | Агрессивное ограничение — снижение параллелизма         |
| **90°C** (Выключение) | Полная остановка обработки на GPU — требуется охлаждение |

{% hint style="warning" %}
**Обеспечьте адекватную вентиляцию и отвод тепла** для непрерывной обработки, особенно в закрытых полевых корпусах или бортовых системах. Термическое ограничение снизит пропускную способность обработки для защиты аппаратного обеспечения.
{% endhint %}

***

## Управление памятью

Устройства Jetson используют **объединенную память** — GPU и CPU используют одну и ту же физическую оперативную память. Это означает, что указанный объем VRAM (например, 15,3 ГБ на Orin NX 16 ГБ) не является выделенной памятью GPU; он используется совместно с операционной системой и другими процессами.

### Рекомендации по подкачке

Для больших наборов данных или обработки с дебайеризацией с учетом текстур (Texture Aware) Chloros может рекомендовать создать пространство подкачки:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Ориентировочный объем памяти на одно изображение:**

* Стандартная дебайеризация: ~10 МБ на изображение
* Дебайеризация с учетом текстур: ~15 МБ на изображение

Chloros автоматически рассчитывает необходимый объем памяти на основе размера вашего набора данных и предупреждает вас, если рекомендуется использовать подкачку.

### Резервный вариант при нехватке памяти (OOM)

Если во время обработки обнаруживается нехватка памяти:

1. Chloros автоматически уменьшает количество рабочих процессов на GPU
2. Переключается с конвейера `fused_gpu` на `tiled_gpu` (более эффективный с точки зрения использования памяти)
3. Продолжает обработку с пониженной пропускной способностью, а не завершает работу

***

## Развертывание в полевых условиях

### Рекомендации по энергопотреблению

| Модель Jetson     | Типичное энергопотребление | Примечания                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5–10 Вт              | USB-C или разъем типа «barrel»    |
| Jetson Orin Nano | 7–15 Вт              | Разъем DC barrel          |
| Jetson Orin NX   | 10–25 Вт             | Разъем DC barrel          |
| Jetson AGX Orin  | 15–60 Вт             | USB-C PD или разъем DC barrel |

Рассчитайте потребность в питании для непрерывной обработки — пиковое энергопотребление наблюдается во время выполнения потока 3 (обработка), требующего интенсивной работы графического процессора.

### Рекомендации по хранению

* **SSD NVMe** настоятельно рекомендуется для развертываний на arm64
* SD-карты слишком медленны для обработки — используйте их только в качестве загрузочного носителя
* Запланируйте объем, в 2–3 раза превышающий размер исходных изображений, для обработанных выходных данных

### Работа без монитора через SSH

Chloros CLI идеально подходит для бездисплейных развертываний Jetson:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Автоматическая обработка с помощью systemd

Создайте службу systemd для автоматической обработки:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

Свяжите с таймером systemd для запланированной обработки:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Примеры рабочих процессов

### Базовая обработка на Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK на Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Пакетная обработка нескольких полетов

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Рекомендуемые системы Jetson для полевого использования

Для полевого и воздушного применения рассмотрите следующие варианты несущих плат Jetson Orin NX 16 ГБ:

* **Воздушное применение/дроны**: системы с виброустойчивостью (MIL-STD), легкие (менее 300 г), с пассивным охлаждением
* **Условия полевой эксплуатации**: водонепроницаемые корпуса с защитой IP67/IP69K и подключением к камере GigE по PoE
* **Минимальная/бюджетная**: наборы для разработчиков с дополнительными корпусами

Обратитесь в [MAPIR службу поддержки](https://www.mapir.camera/community/contact) для получения конкретных рекомендаций по аппаратному обеспечению для вашего сценария развертывания.

***

## Следующие шаги

* [Linux Установка](linux-installation.md) — Общие сведения об установке Linux
* [Динамическая адаптация вычислительных ресурсов](../processing-architecture/dynamic-compute-adaptation.md) — Полное руководство по стратегиям вычислений
* [Конвейер обработки](../processing-architecture/processing-pipeline.md) — Описание 4-поточного конвейера
* [CLI : Командная строка](../CLI.md) — Полное руководство по CLI
* [API : Python SDK](../api-python-sdk.md) — Полное руководство по SDK
