# BI.ZONE Triage
Репозиторий содержит [BI.ZONE Triage](https://bi.zone/catalog/products/triage/) - это бесплатный инструмент для автоматического сбора данных с хостов при проведении криминалистического анализа и поиска индикаторов компрометации. Инструмент создан на базе решения [BI.ZONE EDR](https://bi.zone/catalog/products/edr/).

# Описание
**BI.ZONE Triage поможет:**
* собрать данные с хоста, такие как: информация о системе, сетевой активности, историю выполнения команд и важные логи. Для этого используются Profiles или Presets;
* просканировать YARA файлы и активные процессы. Подробнее в разделе **Сканирование Yara**.

## Использование BI.ZONE Triage
Запуск утилиты возможен только с root привелегиями.

### Аргументы командной строки
```
  -h, --help           Вывести help по использованию утилиты
  --version            Вывести версию утилиты

Параметры запуска:
  -p=PROFILE..., --profiles
                       Указать список желаемых профайлов/пресетов через запятую
                       Указать 'list' для просмотра доступных профайлов и пресетов
                       Для исключения профайла используйте префикс '-' (подробнее в примерах использования)
  -c=CONFIG, --config  Путь к своему конфигурационному файлу установить в CONFIG
  --customdir=DIR      Путь к директории для профайла 'customfolderinfo' установить в DIR

  сканирование yara:
    --yararules=YARAFILE
                       Путь к файлу с yara правилами установить в YARAFILE
    --yaradir=YARADIR  Путь к файлу или директории для сканирования yara установить в YARADIR
    --yarapid=YARADPID Перечислить PID процессов для сканирования yara в YARAPID через запятую


Выходные параметры:
  директория для сохранения результатов работы:
    -o=DIR, --outdir   Установить путь к директории для сохранения результатов работы в DIR
                       (по умолчанию сохраняется в директорию где расположен bin файл утилиты)

  адрес для отправки логов по сети:
    --host=ADDR        Установить адрес (IP или доменное имя) для отправки данных в ADDR
    --port=PORT        Установить порт для отправки данных в PORT

  архивирование:
    -z, --zip          Pack the report in a Zip archive
    --zipname=ZIPNAME  Set the Zip archive file name to ZIPNAME (ZIPNAME.zip if the extension is not specified???)
    --zippwd=PWD       Set the Zip archive password to PWD


Дополнительные опции:
  -t=DIR, --tempdir    Set the path to the directory for temporary files to DIR
  --utc                Use the UTC timezone. By default, the local timezone is used
  --limit-time=SEC     Set the time limit (in seconds) to SEC
  --limit-ram=RAMPCNT  Set the RAM limit (in percent) to RAMPCNT
  --limit-cpu=CPUPCNT  Set the CPU limit (in percent) to CPUPCNT
```
### Profiles and Presets
Для сбора данных в решении предусмотрено использование Profiles и Presets (ключ ```-p``` или ```--profiles```).
Для вывода всех доступных Profiles и Presets используйте: ```-p list```

**Profiles включают в себя следующие наборы данных:**
- ```hostinfo```: данные о хосте (host name, OS info, CPU info, etc);
- ```networks```: информация об IP адресах системы, DNS серверах;
- ```netconn```: активные сетевые подключения; открытые на прослушивание порты;
- ```processes```: инвентаризация запущенных процессов;
- ```sessions```: активные пользовательские сессии;
- ```users```: пользователи; группы; SSH-ключи; правила sudoers;
- ```autoruns``` (данные автозагрузки): cron; at; systemd-сервисы; systemd-таймеры; модули ядра; LD_PRELOAD; shell-скрипты (.bashrc, .bash_profile, etc.);
- ```logonhist```: история попыток входа в систему (wtmp, btmp);
- ```cmdhist```: история вводимых команд (.bash_history и т.п.);
- ```openfiles```: информация об открытых файлах;
- ```shares```: информация об открытых общих ресурсах SMB/NFS;
- ```mounts```: информация о примонтированных носителях и файловых системах;
- ```containers```: Docker-контейнеры (Running, Stopped);
- ```packages```: установленные пакеты;
- ```keyfoldersinfo```: инвентаризация файлов в ключевых директориях системы (tmp, bin, sbin, dev, run, proc и другие) с обогащением атрибутами файловой системы;
- ```customfoldersinfo```: инвентаризация файлов в указанных директориях системы (используется только совместно с ключом: ```--customdir```) с обогащением атрибутами файловой системыю.

**Presets включают наборы Profiles, заранее подготовленные для быстрого сбора данных:**
- ```full```: пресет для быстрого сбора всех доступных в решении Profiles;
- ```investigation```: пресет, необходимый для проведения криминалистического анализа, включает следующие Profiles: hostinfo, processes, netconn, sessions, autoruns, networks, users, logonhist, cmdhist;
- ```iocsfs```: пресет для сбора данных, полезных для проверки на файловые индикаторы компрометации. Включает следующие Profiles: hostinfo, processes, netconn, sessions, autoruns, networks, users, logonhist, cmdhist;
- ```iocsfull```: пресет для сбора данных системы, полезных для проверки на все индикаторы компрометации. Включает следующие Profiles: hostinfo, processes, netconn, sessions, autoruns, networks, users, logonhist, cmdhist;
Пресеты и профайлы при сканировании указываются через запятую, например:
```-p logonhist,cmdhist``` или ```-p investigation,containers```
Для того, чтобы исключить определенные наборы данных из сканирования при запуске Preset, его необходимо указать выбранный для исключения Preset с префиксом '-', например:
```-p full,-autoruns```

## Сканирование Yara
В решении предусмотрен Yara сканер. Для запуска сканирования необходимо подготовить Yara правила.
Для запуска сканирования используются ключи ```--yararules``` и ```--yaradir```, которые позволют просканироваить:
- указанный файл. Для этого укажите в ключе ```--yaradir``` путь к файлу. Например:
```sudo ./bz_triage_nix --yararules ./myrules.yar --yaradir /tmp/sample.pdf```
- указанную папку с рекурсией. Для этого укажите в ключе ```--yaradir``` путь к папке с указанием '*' в конце пути. Например:
```sudo ./bz_triage_nix --yararules ./myrules.yar --yaradir /tmp/*```
- важные области операционной системы. Набор папок, включающий в с себя: tmp, bin, sbin, dev, run, proc и другие.
- активные процессы. Для этого укажите в ключе ```--yarapid``` список PID процессов через запятую. Пример:
```sudo ./bz_triage_nix --yararules ./myrules.yar --yarapid 5514,5617```

Допустимо совместное использование Yara сканера со сбором данных по Presets и Profiles.

## Сохранение результатов работы
При работе утилиты каждое вхождение данных (событие), например: запись cron, пользователь, SSH-ключ, контейнер Docker или примонтированный носитель; сохраняется в виде JSON структуры заданного формата по принципу "одно вхождение — один JSON".
Примеры JSON файлов событий выложены в папке [result examples]().
Описание модели данных событий разных типов представлено в [Wiki]().
Возможно как локальное сохранение результатов работы, так и отправка их по сети в Log Management System.
**Локальное сохранение**
При сборе данных по Profiles или Prosets, собранные данные разбиваются по Profiles и представляются в виде отдельных файлов с сериями событий. Один JSON - одна строка файла.
По умолчанию, полученные файлы сохраняются в текущую директорию, из которой запущена утилита. При необходимости, можно указать собственную директорию через аргумент ```-o``` или ```--output```. Пример:
```sudo ./bz_triage_nix -p investigation,containers -o /home/testuser```
**Отправка результатов в Log Management System**
Для отправки полученных данных в Log Management Sytem необходимо установить атрибуты  ```--host``` и ```--port```. Например:
```sudo ./bz_triage_nix -p investigation,containers --host 10.10.10.10 --port 5000```
**Архивирование результатов**


# Примеры использования
## Сбор данных с пресетом для криминалистического анализа, сканирование Yara указанной папки и отправка данных в ELK
Пример конфигурации Logstash:
Пример запуска BI.ZONE Triage:
```
sudo ./bz_triage_nix -p investigation --yararules ./rules.yar --yaradir /tmp/* --host logstash.myhost.ru --port 5000 
```
Скриншоты:
