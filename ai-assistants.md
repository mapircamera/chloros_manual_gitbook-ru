# Использование Chloros с ИИ-помощниками

Данное руководство предназначено для двух категорий пользователей: людей и ИИ-помощников, с помощью которых люди всё чаще выполняют свою работу. На каждой странице приводятся точные значения, значения по умолчанию и команды, готовые к копированию и вставке, чтобы помощник (Claude, ChatGPT, Copilot, агент по программированию и т. д.) мог с первого раза написать рабочую автоматизацию Chloros.

Версия Chloros: **

1.2.0**. Платформы CLI/SDK: Windows 10/11 x64 и Linux (x86_64 / Jetson aarch64).

## Что нужно передать вашему помощнику

| Ресурс | URL | Для чего он нужен |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Машинно-читаемый указатель всех страниц данного руководства. |
| **CLI Справочник** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | Полный набор команд `chloros-cli`: все команды, флаги, значения по умолчанию, коды завершения и правила для папки вывода. Написано для использования LLM. |
| **Справочник SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | Полный обзор `chloros_sdk` Python API: классы, сигнатуры, исключения и практические примеры. Написано для использования в LLM. |
| **Любая страница в виде необработанного Markdown** | добавьте `.md` к странице URL | например, `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` возвращает страницу в виде исходного Markdown — идеально подходит для вставки в контекстное окно или извлечения из агента. |

Ссылки внутри руководства: [CLI Справочник](reference/cli-reference.md) · [Справочник по SDK](reference/sdk-reference.md).

{% hint style="info" %}
Эти две справочные страницы являются самодостаточными: помощнику, ознакомившемуся с одной из них, не понадобится остальная часть руководства для написания правильного скрипта.
{% endhint %}

## Готовые рецепты

Скопируйте, заполните форму `<placeholders>` и вставьте в свой помощник.

### 1. Обработка папки с данными полёта в NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Пакетный мониторинг каталога записей

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Подключение массива LATTICE и запись данных

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

Каждый из этих примеров — реальное, проверенное поведение Chloros 1.2.0 — они устраняют классические причины сбоев в автоматизации, написанной машиной:

* **Отсутствие сложных настроек.**Помощники интеллектуального подключения SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) и точки входа обработки (`ChlorosLocal`, `process_folder`)**автоматически запускают локальный бэкенд**. Сгенерированному скрипту не требуется открытый графический интерфейс или запущенный вручную сервер — необходимо лишь, чтобы на рабочем столе был установлен пакет desktop/CLI.
* **Весь конвейер представляет собой один вызов.** `chloros_sdk.process_folder("path", indices=["NDVI"])` выполняет импорт → калибровку → расчет коэффициента отражения → экспорт индекса от начала до конца. Меньше поверхностей, меньше точек, где сгенерированный скрипт может дать сбой.
* **Запуски без вывода результатов проходят самодиагностику.** После завершения работы `process()` к результату прилагается сводка запуска, а каждое пояснение по обработке (например, *почему* запуск не дал результата) также повторно выводится в виде Python `UserWarning` — так что даже скрипт, который никогда не проверяет словарь результатов, отобразит диагноз.
* **CLI завершается с явным сбоем.**Запуск с кодом `chloros-cli process`, который запрашивал результаты, но не вывел ни одного, выводит `Processing finished but wrote no image products.` и**завершается с ненулевым кодом**, поэтому скрипты оболочки и CI обнаруживают это с помощью простой проверки кода завершения. Успешные запуски возвращают код `Image products written: N`.

Одна асимметрия, о которой должен знать ассистент: `process()` из SDK намеренно **не** генерирует исключение при запуске с нулевым количеством продуктов — вместо этого он сообщает об этом через сводку/подсказки. Если конвейер Python должен остановиться при пустом запуске, проверьте сводку (рецепт 2 это делает).

## Предостережения

* **Требуется вход в систему с учетной записью Chloros+.**Для CLI и SDK требуется**платный** уровня Chloros+, что контролируется на стороне сервера: запросы завершаются с ошибкой `401 AUTH_REQUIRED` при отсутствии входа в систему и с ошибкой `403 PLAN_UPGRADE_REQUIRED` на бесплатном уровне. Запустите `chloros-cli login` один раз на каждом компьютере перед запуском сгенерированных скриптов. См. [Chloros+ Вход в систему](chloros+-login.md).
* **Команды захвата управляют реальным оборудованием.** Команды `lattice` / `daq` / `project` и объекты сеанса SDK обеспечивают подключение, потоковую передачу данных и запуск физических камер и датчиков. Перед первым запуском проверьте сгенерированный скрипт и запустите его в присутствии оператора оборудования.
* **Проведите выборочную проверку выходных данных.** Перед публикацией результатов проверьте папки с результатами и несколько значений пикселей. В частности, TIFF-файлы с данными отражательной способности масштабируются в зависимости от источника — ознакомьтесь с XMP-тегом `Chloros:PixelScale` (LATTICE: 32768 = отражательная способность 1,0; Survey3: 65535), а не предполагайте делитель. Оба справочных документа описывают это в разделе «Чтение пикселей отражательной способности».
* **Небольшие подводные камни, которые сбивают с толку сгенерированный код:**`pool-record` записывает в файловую систему**хоста бэкэнда** (по умолчанию `~/Documents/DAQ Live View/`); на машинах с несколькими сетевыми интерфейсами предпочтительнее использовать `daq pool-connect --eth-host <ip-or-hostname>` вместо автоматического определения; и используйте `http://127.0.0.1:5000` (ни в коем случае не `localhost`) везде, где встречается бэкенд URL.
