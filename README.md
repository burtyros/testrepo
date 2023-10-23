# BI.ZONE Triage
Репозиторий содержит [BI.ZONE Triage](https://bi.zone/catalog/products/triage/) - это бесплатный инструмент для автоматического сбора данных с хостов при проведении криминалистического анализа и поиска индикаторов компрометации. Инструмент создан на базе решения [BI.ZONE EDR](https://bi.zone/catalog/products/edr/).

# Описание
**BI.ZONE Triage поможет:**
* собрать данные с хоста, такие как: информация о системе, сетевой активности, историю выполнения команд и важные логи. Для этого используются Profiles или Presets;
* просканировать YARA файлы и активные процессы. Подробнее в разделе **Сканирование Yara**.
Список аргументов
## Profiles and Presets
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
