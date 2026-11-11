# Использование Chloros с ИИ-помощниками

Данное руководство предназначено для двух аудиторий: людей и ИИ-помощников, с которыми люди все чаще работают. На каждой странице приведены точные значения, значения по умолчанию и команды, готовые к копированию и вставке, чтобы помощник (Claude, ChatGPT, Copilot, агент для программирования и т. д.) мог с первого раза написать рабочую автоматизацию Chloros.

Версия Chloros: **

1.3.0**. Платформы CLI/SDK: Windows 10/11 x64 и Linux (x86_64 / Jetson aarch64).

## Что предоставить вашему помощнику

| Ресурс | URL | Для чего он нужен |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Машинно-читаемый индекс всех страниц данного руководства. |
| **Справочник по CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Полный набор команд `chloros-cli`: все команды, флаги, значения по умолчанию, коды завершения и правила для папки вывода. Написано для использования LLM. |
| **Справочник по SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Полный Python-API `chloros_sdk`: классы, сигнатуры, исключения и практические примеры. Написано для использования LLM. |
| **Любая страница в виде необработанного Markdown** | добавьте `.md` к URL-адресу страницы | например, `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` возвращает страницу в виде необработанного Markdown — идеально подходит для вставки в окно контекста или получения от агента. |

Ссылки внутри руководства: [Справочник по CLI](reference/cli-reference.md) · [Справочник по SDK](reference/sdk-reference.md).

{% hint style="info" %}
Эти две справочные страницы являются самодостаточными: помощнику, ознакомившемуся с одной из них, не понадобится остальная часть руководства для написания правильного скрипта.
{% endhint %}

## Позвольте вашему помощнику запускать Chloros напрямую: `chloros-mcp`

Во всех приведенных выше примерах ваш помощник *пишет* скрипт или команду. С помощью `chloros-mcp` помощник, поддерживающий [Model Context Protocol](https://modelcontextprotocol.io) (MCP), например Claude Desktop, Claude Code или Cursor, управляет самим Chloros. Вы задаете запрос простым языком, и он вызывает инструменты сервера:

* **Обработка изображений:** создание проекта, импорт папки с снимками, выбор настроек, обработка и отчет о результатах экспорта.
* **Управление оборудованием:** подключение камер LATTICE, матриц и световых датчиков DAQ, предварительный просмотр, настройка экспозиции и цветопередачи, съемка, запись и обновление прошивки.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

Инструменты сервера охватывают функционал [Python SDK](api-python-sdk.md). Они устанавливаются вместе с SDK, поэтому обычно ничего больше устанавливать не нужно: в Windows установщик Chloros помещает `chloros-mcp.exe` в папку `Scripts` вашего Python, а в Linux `.deb` помещает файл `chloros-mcp` в папку `/usr/bin`.

**Код Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**добавьте это в файл `claude_desktop_config.json` (откройте его через**Настройки > Разработчик > Редактировать конфигурацию**), затем перезапустите Claude Desktop.**Cursor** принимает ту же запись в файле `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Если клиент сообщает, что сервер не запустился, вероятно, `chloros-mcp` отсутствует в списке `PATH`, который он видит: укажите полный путь (`where chloros-mcp` в Windows, `which chloros-mcp` в Linux).

На том же компьютере должен быть установлен Chloros, и вы должны войти в систему — в приложении или с помощью `chloros-cli login` — на любом платном тарифе Chloros+ (Copper или выше). Сервер не может войти в систему за вас.

{% hint style="info" %}
Настройка клиента для каждого приложения, параметры, все инструменты и поведение процесса: [MCP Server (AI Assistants)](reference/sdk-reference.md#mcp-server-ai-assistants) в справочнике по SDK.
{% endhint %}

## Шаблоны запросов

Скопируйте, заполните `<placeholders>` и вставьте в своего помощника.

### 1. Преобразование папки с данными съёмки в NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Пакетный мониторинг каталога снимков

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Подключение массива LATTICE и съемка

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Запись спектров светового датчика DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
Скрипты DAQ из командной строки всегда проходят через семейство `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Другие подкоманды `daq`, которые может придумать ваш помощник, недоступны в поставляемых сборках и приводят к завершению работы с ошибкой.
{% endhint %}

## Почему скрипты, написанные ИИ, хорошо работают с Chloros

Каждый из этих примеров — реальное, проверенное поведение Chloros 1.3.0 — они устраняют классические причины сбоев в автоматизации, написанной машиной:

* **Отсутствие сложных настроек.**Помощники интеллектуального подключения SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) и точки входа обработки (`ChlorosLocal`, `process_folder`)**автоматически запускают локальный бэкенд**. Сгенерированному скрипту не требуется открытый графический интерфейс или запущенный вручную сервер — достаточно лишь установленного пакета для рабочего стола или CLI.
* **Весь конвейер представляет собой один вызов.** `chloros_sdk.process_folder("path", indices=["NDVI"])` выполняет импорт → калибровку → расчет коэффициента отражения → экспорт индекса от начала до конца. Меньше поверхностных элементов — меньше мест, где сгенерированный скрипт может дать сбой.
* **Запуски без вывода результатов проходят самодиагностику.** После завершения работы `process()` к результату прилагается сводка запуска, а каждое указание по обработке (например, *почему* запуск не дал результата) также повторно выводится в виде Python-кода `UserWarning` — так что даже скрипт, который никогда не проверяет словарь результатов, отобразит диагноз.
* **CLI выдаёт явную ошибку.**Запуск `chloros-cli process`, который запрашивал результаты, но не записал ни одного, выводит `Processing finished but wrote no image products.` и**завершается с ненулевым кодом**, поэтому скрипты оболочки и CI обнаруживают это с помощью простой проверки кода завершения. Успешные запуски возвращают код `Image products written: N`, а также `(plus M clamp masks)`, если были записаны маски. Команды захвата аналогичным образом указывают каждый файл, который не удалось записать, и завершают работу с кодом `1`.
* **Относительные пути интерпретируются так, как того ожидает скрипт.** При использовании локального бэкэнда относительный путь вывода (`-o out/` в CLI, `output_dir="out/"` в SDK) преобразуется относительно папки, из которой запускается команда или скрипт.

Одна асимметрия, о которой должен знать помощник: `process()` в SDK намеренно **не** генерирует исключение при запуске без результатов — вместо этого он сообщает об этом через сводку/подсказки. Если конвейер Python должен останавливаться при пустом запуске, проверьте сводку (рецепт 2 это делает).

## Предостережения

* **Требуется вход в Chloros+.**Для работы с CLI и SDK требуется**платный** тариф Chloros+, что проверяется на стороне сервера: запросы завершаются с ошибкой `401 AUTH_REQUIRED` при отсутствии входа в систему и с ошибкой `403 PLAN_UPGRADE_REQUIRED` на бесплатном тарифе. Перед запуском сгенерированных скриптов выполните команду `chloros-cli login` один раз на каждом компьютере. См. [Вход в Chloros+](chloros+-login.md).
* **Команды захвата управляют реальным оборудованием.** Команды `lattice` / `daq` / `project`, объекты сеанса SDK и инструменты `chloros-mcp` обеспечивают подключение, потоковую передачу данных и запуск физических камер и датчиков. Перед первым запуском проверьте сгенерированный скрипт и запустите его в присутствии оператора, отвечающего за оборудование.
* **Проведите выборочную проверку результатов.** Перед публикацией результатов проверьте папки с данными и несколько значений пикселей. В частности, TIFF-файлы с данными отражательной способности масштабируются в зависимости от источника — ознакомьтесь с тегом XMP `Chloros:PixelScale` (файлы LATTICE содержат его: 32768 = 1,0 отражательная способность; файлы Survey3 его не содержат и используют 65535 = 1,0) вместо того, чтобы предполагать делитель. Оба справочных документа описывают это в разделе «Чтение пикселей отражательной способности».
* **Небольшие подводные камни, которые могут сбить с толку сгенерированный код:**`pool-record` записывает в файловую систему**хоста бэкэнда** (по умолчанию — `~/Documents/DAQ Live View/`); на машинах с несколькими сетевыми интерфейсами предпочтительнее использовать `daq pool-connect --eth-host <ip-or-hostname>` вместо автоматического обнаружения; используйте `http://127.0.0.1:5000` (ни в коем случае не `localhost`) везде, где встречается URL-адрес бэкэнда; скрипт, проходящий по дереву выходных файлов проекта, должен пропускать папки `clamp_masks/` (их файлы `*_clampmask.tif` представляют собой маски, а не изображения — см. [Форматы выходных изображений](output-image-formats.md#clamp-masks)); а в Windows, где отключены длинные пути, папки проектов должны быть короткими — папка проекта, длина пути к которой превышает примерно 130 символов, отклоняется до начала обработки.
