# BI.ZONE Triage
Репозиторий содержит [BI.ZONE Triage](https://bi.zone/catalog/products/triage/) - это бесплатный инструмент для автоматического сбора данных с хостов при проведении криминалистического анализа и поиска индикаторов компрометации. Инструмент создан на базе решения [BI.ZONE EDR](https://bi.zone/catalog/products/edr/).

# Описание
**BI.ZONE Triage поможет:**
* собрать данные с хоста, такие как: информация о системе, сетевой активности, историю выполнения команд и важные логи. Для этого используются Profiles или Presets;
* просканировать YARA файлы и активные процессы. Подробнее в разделе **Сканирование Yara**.
Собранные данные сохраняются плоские файлы формата JSON.

## Использование BI.ZONE Triage
Запуск утилиты возможен только с root привелегиями.

### Аргументы командной строки
```
  -h, --help           Вывести help по использованию утилиты
  --version            Вывести версию утилиты

Параметры запуска:
  -p=PROFILE..., --profiles
                       Specify the list of desired profiles and/or presets separated by commas
                       Specify 'list' to print the available values and exit
                       To exclude a profile, use the '-' prefix (see examples)
  -c=CONFIG, --config  Set the path to a custom configuration file to CONFIG
  --customdir=DIR      Set the directory for the 'customfolderinfo' profile to DIR

  сканирование yara:
    --yararules=YARAFILE
                       Set the path to the yara rules feed to YARAFILE
    --yaradir=YARADIR  Set the directory to scan by yara to YARADIR
    --yarapid=YARADPID Set the PID to scan by yara to YARAPID


Выходные параметры:
  директория для сохранения результатов работы:
    -o=DIR, --outdir   Set the path to the output directory for saving the report to DIR
                       (by default, it is the current working directory)

  адрес для отправки логов по сети:
    --host=ADDR        Set the host address (IP address or domain) for sending the report to ADDR
    --port=PORT        Set the port for sending the report to PORT

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
- hostinfo:
  - данные о хосте (host name, OS info, CPU info, etc);
- networks:
  - информация об IP адресах системы, DNS серверах;
- netconn:
  - активные сетевые подключения;
  - открытые на прослушивание порты;
- processes:
  - инвентаризация запущенных процессов;
- sessions:
  - активные пользовательские сессии;
- users:
  - пользователи;
  - группы;
  - SSH-ключи;
  - правила sudoers;
- autoruns (данные автозагрузки):
  - cron;
  - at;
  - systemd-сервисы;
  - systemd-таймеры;
  - модули ядра;
  - LD_PRELOAD;
  - shell-скрипты (.bashrc, .bash_profile, etc.);
- logonhist:
  - история попыток входа в систему (wtmp, btmp);
- cmdhist:
  - история вводимых команд (.bash_history и т.п.);
- openfiles:
  - информация об открытых файлах;
- shares:
  - информация об открытых общих ресурсах SMB/NFS;
- mounts:
  - информация о примонтированных носителях и файловых системах;
- containers:
  - Docker-контейнеры (Running, Stopped);
- packages:
  - установленные пакеты;
- keyfoldersinfo:
  - инвентаризация файлов в ключевых директориях системы (tmp, bin, sbin, dev, run, proc и другие) с обогащением атрибутами файловой системы;
- customfoldersinfo:
  - инвентаризация файлов в указанных директориях системы (используется только совместно с ключом: ```--customdir```) с обогащением атрибутами файловой системыю.

**Presets включают наборы Profiles, заранее подготовленные для быстрого сбора данных:**
* full: пресет для быстрого сбора всех доступных в решении Profiles;
* investigation: пресет, необходимый для проведения криминалистического анализа, включает следующие Profiles: hostinfo, processes, netconn, sessions, autoruns, networks, users, logonhist, cmdhist;
* iocsfs: пресет для сбора данных, полезных для проверки на файловые индикаторы компрометации. Включает следующие Profiles: hostinfo, processes, netconn, sessions, autoruns, networks, users, logonhist, cmdhist;
* iocsfull: пресет для сбора данных системы, полезных для проверки на все индикаторы компрометации. Включает следующие Profiles: hostinfo, processes, netconn, sessions, autoruns, networks, users, logonhist, cmdhist.

Пресеты и профайлы при сканировании указываются через запятую, например:
```-p logonhist,cmdhist``` или ```-p investigation,containers```

Для того, чтобы исключить определенные наборы данных из сканирования при запуске Preset, его необходимо указать выбранный для исключения Preset с префиксом '-', например:
```-p full,-autoruns```

## Сканирование Yara
В решении предусмотрен Yara сканер. Для запуска сканирования необходимо подготовить Yara правила.
Для запуска сканирования используются ключи ```--yararules``` и ```--yaradir```, которые позволют просканироваить:
* указанный файл. Для этого укажите в ключе ```--yaradir``` путь к файлу;
* указанную папку с рекурсией. Для этого укажите в ключе ```--yaradir``` путь к папке с указанием '*' в конце пути. Например: ```--yaradir /tmp/*```
* важные области операционной системы. Набор папок, включающий в с себя: tmp, bin, sbin, dev, run, proc и другие.
* активные процессы.

# Примеры использования
**Собрать данные по всем доступным Profiles и сохранить результаты в указанную папку:**
```
sudo ./bz_triage_nix -p full -o ./scanresults 
```
**Выполнить**
