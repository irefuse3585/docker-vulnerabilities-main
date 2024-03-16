# Подготовка среды

Настроим docker-compose файл, в котором в качестве сервисов будет: postgres и server. Настроим постоянное хранение, сетевое окружение и healthckecks. Для server настроим Dockerfile с необходимой версией питона (облегченной) и необходимыми библиотеками. В app.py находится логика приложения: при помощи Flask будет подниматься сервер, из postgres будем получать ФИО, формировать HTML страницу для возврата.

Сделаем сборку server  
   <img width="981" alt="Снимок экрана 2024-03-16 в 15 10 05" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/bd1df931-6116-43a4-ab60-23aa7341a6e3">
   
С помощью команды docker-compose up -d развернем сервисы. В браузере по адресу localhost увидим результат
   <img width="986" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/0b544a26-d45f-4bd5-892b-2e46ad06976c">
# Trivy

Установим trivy.

<img width="976" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/547d3429-2388-4e06-b00f-0f6f3fe80bd2">

Запустим сканирование. В начале произойдет загрузка БД с уязвимостями. Далее появятся найденные уязвимости.

<img width="987" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/3755c552-b0fb-4586-8da5-c0ca96712a50">

Результат сканирования:

**Python (python-pkg)**
  
В библиотеке pip (METADATA) найдена одна уязвимость:

1. CVE-2023-5752 - При установке пакета по URL-адресу Mercurial VCS (т. е. «pip install hg+...») с помощью pip до версии v23.3 указанная версия Mercurial может использоваться для внедрения произвольных параметров конфигурации в вызов «hg clone» (т. е. « --конфигурация"). Управление конфигурацией Mercurial может изменить способ и тип установки репозитория. Эта уязвимость не затрагивает пользователей, которые выполняют установку не из Mercurial.

Решение: уязвимость существует до версии 23.3, необходимо использовать библиотеку, начиная с версии 23.3. (написать Dockerfile, где прописать обновление)

# bench-security

В отдельную папку склонируем репозиторий утилиты bench-security

<img width="594" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/8d9d7add-8f71-4102-a78e-07b1f15ccc20">

Запустим ее:

<img width="698" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/1a402789-4401-43cd-8c0c-676707c15479">

Получим счет:

<img width="144" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/2ba648ae-685c-4553-bc74-6a03581c0f62">

В результате работы программы видим следующие предупреждения:

**1.1.1 - Ensure a separate partition for containers has been created (Automated) - "Убедитесь, что отдельное разделение для контейнеров было создано (Автоматизировано)."**

<img width="621" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/71179906-0f0e-4324-b067-5b819f94fc93">

Степень риска: Это предупреждение связано с безопасностью и изоляцией контейнеров. Отсутствие отдельного разделения может увеличить вероятность воздействия компрометированного контейнера на другие системы или контейнеры.

Как исправить:
1. Настройка отдельного разделения: Создайте отдельный раздел для контейнеров, чтобы изолировать их работу от других систем. Это поможет предотвратить распространение проблем между контейнерами.
2. Использование инструментов управления: Воспользуйтесь инструментами управления контейнерами, которые позволяют настроить изоляцию и безопасность контейнеров, такими как Docker Compose или Kubernetes.

**1.1.3 - Ensure auditing is configured for the Docker daemon (Automated) - "Убедитесь, что настроен аудит для демона Docker (Автоматизировано)."**

<img width="557" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/bf4a2094-3fef-4ae1-b973-b3c2e6a4e9c9">

Степень риска: Отсутствие настроенного аудита для демона Docker может привести к неспособности отслеживать события, связанные с операциями Docker, что создает слепые пятна в безопасности и возможность упущения аномалий или атак.

Как исправить:
1. Настройка журналирования: Настройте аудит для демона Docker, чтобы записывать события, связанные с его работой. Это включает операции, авторизации, аутентификацию и другие важные действия.
2. Анализ и мониторинг: Организуйте мониторинг и анализ журналов аудита Docker, чтобы быстро обнаруживать подозрительную активность или нарушения безопасности.

**1.1.4 - Ensure auditing is configured for Docker files and directories -/run/containerd (Automated) - "Убедитесь, что настроен аудит для файлов и каталогов Docker - /run/containerd (Автоматизировано)."**

<img width="750" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/16890514-30b0-44ea-8d93-de36e49a2ef8">

Степень риска: Не настроенный аудит для файлов и каталогов Docker, таких как /run/containerd, означает, что возможны незамеченные изменения, атаки на файловую систему или несанкционированный доступ к системным ресурсам.

Как исправить:
1. Настройка аудита: Настройте аудит для файлов и каталогов, связанных с Docker, чтобы учитывать изменения, доступ и другие операции.
2. Мониторинг и анализ: Установите мониторинг на файловые системы Docker для выявления подозрительной активности или нарушений безопасности.

**2.2 - Ensure network traffic is restricted between containers on the default bridge (Scored) - "Убедитесь, что сетевой трафик ограничен между контейнерами на стандартном мосту (Оценено)."**

<img width="704" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/30aad4b3-6e9e-4798-8329-949fdb680e4c">

Степень риска: Ограничение сетевого трафика между контейнерами на стандартном мосту крайне важно для предотвращения несанкционированного доступа и распространения возможных угроз внутри контейнерной среды. Отсутствие такого ограничения может привести к уязвимостям в безопасности и утечкам данных между контейнерами.

Как исправить:
1. Сетевые политики: Настройте правила сетевого доступа и политики в вашей контейнерной среде для блокировки нежелательного трафика между контейнерами.
2. Использование сетевых абстракций: Рассмотрите использование сетевых абстракций и инструментов, таких как Docker network plugins или Kubernetes network policies, для более гранулированного управления сетевым трафиком между контейнерами.

**2.9 - Enable user namespace support (Scored) - "Включите поддержку пространств имен пользователей (Оценено)."**

<img width="363" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/3da7f04d-7ad2-48f5-a5b8-820a9c5d9851">

Степень риска: Отсутствие поддержки пространств имен пользователей (user namespace) может повысить риск компрометации безопасности контейнеров, так как это позволяет более гранулированно управлять разрешениями и доступом пользователей внутри контейнеров. Без этой функциональности атакующий может иметь повышенные привилегии в случае успешной атаки.

Как исправить:
1. Включение поддержки: Активируйте поддержку пространств имен пользователей в конфигурации Docker или другой контейнерной платформы.
2. Конфигурация пространств имен: Настройте подходящие параметры и политики для использования пространств имен пользователей в соответствии с требованиями вашей безопасности.

**2.12 - Ensure that authorization for Docker client commands is enabled (Scored) - "Убедитесь, что авторизация для команд клиента Docker включена (Оценено)."**

<img width="607" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/c3fa5dde-ed1a-452e-b69d-dd7b45a30966">

Степень риска: Неактивированная авторизация для команд клиента Docker может привести к возможности выполнения несанкционированных операций или изменений в контейнерах. Это создает риск для безопасности и целостности контейнерной среды.

Как исправить:
1. Настройка авторизации: Включите и настройте механизм авторизации для команд клиента Docker, чтобы контролировать доступ и операции.
2. Использование ролевых прав: Работайте с ролевыми правами и доступом, чтобы определить, какие пользователи или группы могут выполнять определенные операции.

**2.13 - Ensure centralized and remote logging is configured (Scored) - "Убедитесь, что настроено централизованное и удаленное журналирование (Оценено)."**

<img width="533" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/bc3cb34d-d9ad-4a3e-8881-17f1a1c6422e">

Степень риска: Отсутствие настроенного централизованного и удаленного журналирования может затруднить обнаружение и реакцию на события безопасности или проблемы в контейнерной среде. Это также усложняет мониторинг и анализ действий в контейнерах.

Как исправить:
1. Конфигурация журналирования: Настройте систему централизованного и удаленного журналирования, чтобы записывать и анализировать логи событий в контейнерах.
2. Резервное копирование логов: Убедитесь, что система журналирования настроена на регулярное резервное копирование логов для сохранения целостности данных.

**2.14 - Ensure containers are restricted from acquiring new privileges (Scored) -  "Убедитесь, что контейнерам запрещено приобретать новые привилегии (Оценено)."**

<img width="602" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/d0993017-d9a1-4d49-a3c7-b030954a291f">

Степень риска: Если контейнерам разрешено приобретать новые привилегии, это может увеличить уязвимости и повысить риск атак на вашу контейнерную среду. Неправильная конфигурация привилегий может привести к расширенным возможностям атаки и компрометации безопасности.

Как исправить:
1. Ограничение привилегий: Настройте контейнеры таким образом, чтобы они не могли приобретать новые привилегии во время выполнения.
2. Использование SELinux или AppArmor: Рассмотрите использование инструментов обязательного доступа, таких как SELinux или AppArmor, для управления привилегиями контейнеров.

**2.15 - Ensure live restore is enabled (Scored) - "Убедитесь, что включена возможность восстановления контейнеров в реальном времени (Оценено)."**

<img width="377" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/d2170f28-77d5-4baf-b49d-f5209f93d340">

Степень риска: Отсутствие включенной опции восстановления контейнеров в реальном времени (live restore) может привести к потере данных в случае сбоя или перезапуска Docker демона во время работы контейнеров. Это может вызвать проблемы с непрерывностью работы и потерю состояния приложений.

Как исправить:
Включение live restore: Убедитесь, что опция восстановления контейнеров в реальном времени включена в конфигурации Docker.

**2.16 - Ensure Userland Proxy is Disabled (Scored) - "Убедитесь, что Userland Proxy отключен (Оценено)."**

Степень риска: Включенный Userland Proxy может стать потенциальной точкой уязвимости и увеличить атаку на контейнеры, так как он делегирует сетевую обработку от ядра операционной системы к пользовательскому пространству, что может стать зоной риска.

<img width="396" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/8cadcf7f-4608-4a72-b7f9-c0c148cd8d3b">

Как исправить:
1. Отключение Userland Proxy: Убедитесь, что Userland Proxy отключен в настройках Docker для улучшения производительности и безопасности.
2. Использование режима проксирования: Рассмотрите использование других режимов проксирования (например, iptables) для обеспечения эффективного и безопасного сетевого взаимодействия.

**3.15 - Ensure that the Docker socket file ownership is set to root:docker (Automated) - "Убедитесь, что владелец файла сокета Docker установлен на root:docker (Автоматизировано)."**

<img width="654" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/0314f737-09b6-4178-8a0a-13199940b32b">

Степень риска: Неправильные права доступа к файлу сокета Docker (/var/run/docker.sock) могут повлечь за собой уязвимости и потенциальные атаки на систему через Docker API. Необходимо исправить владельца файла для обеспечения безопасности.

Как исправить:
1. Изменение владельца файла сокета: Выполните команду chown root:docker /var/run/docker.sock для установки правильных прав доступа к файлу сокета Docker.
2. Перезапуск Docker сервиса: После изменения прав на файл сокета, перезапустите сервис Docker, чтобы применить изменения.

**3.16 - Ensure that the Docker socket file permissions are set to 660 or more restrictively (Automated) - "Убедитесь, что права доступа к файлу сокета Docker установлены на 660 или более строго (Автоматизировано)."**

<img width="775" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/ec021f0e-f1ab-4cb8-bb6b-28bc71d13add">

Степень риска: Неправильные права доступа к файлу сокета Docker (/var/run/docker.sock) могут привести к раскрытию конфиденциальной информации, возможности атаки на Docker API и другим безопасностным угрозам. Необходимо скорректировать права доступа к файлу сокета для повышения безопасности.

Как исправить:
1. Изменение прав доступа к файлу сокета: Выполните команду chmod 660 /var/run/docker.sock для установки правильных прав доступа к файлу сокета Docker.
2. Проверка и обновление: Проверьте другие файлы и каталоги на соответствие правилам безопасности и исправьте их при необходимости.

**4.1 - Ensure that a user for the container has been created (Automated) - "Убедитесь, что пользователь для контейнера был создан (Автоматизировано)."**

<img width="553" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/0e5444ce-4f8f-4762-8a01-8fba27f97817">

Степень риска: Использование контейнеров с запущенными процессами от имени пользователя root увеличивает риск возможной компрометации контейнера и всей среды. Запуск процессов от имени привилегированного пользователя может привести к безопасностным уязвимостям и атакам.

Как исправить:
1. Создание пользователей: Создайте отдельных пользователей в контейнерах для запуска процессов с минимальными привилегиями.
2. Правила запуска: Установите правило запуска контейнеров от имени созданных пользователей, а не от имени root.

**4.5 - Ensure Content trust for Docker is Enabled (Automated) - "Убедитесь, что доверие к содержимому для Docker включено (Автоматизировано)."**

<img width="474" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/7a1f5e43-904f-4a42-9b08-992d87e98a24">

Степень риска: Отсутствие включенного доверия к содержимому (Content trust) для Docker может привести к возможным атакам с использованием вредоносных образов контейнеров, изменениям в безопасности и целостности сети Docker.

Как исправить:
1. Включение Content trust: Убедитесь, что доверие к содержимому активировано для подписи и проверки образов перед их запуском.
2. Управление ключами и подписями: Настройте процесс работы с ключами и подписями для обеспечения безопасности контейнеров и образов Docker.

**5.2 - Ensure that, if applicable, an AppArmor Profile is enabled (Automated) - "Убедитесь, что, если это применимо, профиль AppArmor включен (Автоматизировано)."**

<img width="591" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/6ca7fbd5-183d-4276-9e40-b00aa62f0596">

Степень риска: Отсутствие активированного профиля AppArmor для контейнеров (в данном случае для сервера и PostgreSQL) может повлиять на безопасность контейнеров, поскольку AppArmor позволяет управлять доступом и поведением контейнеров в системе.

Как исправить:
1. Создание и применение профиля: Настройте и примените профили AppArmor для контейнеров, чтобы установить правила безопасности и ограничения их действий.
2. Тестирование и адаптация: Проведите тестирование профилей AppArmor для обеспечения корректной работы контейнеров и отсутствия нежелательных блокировок.

**5.3 - Ensure that, if applicable, SELinux security options are set (Automated) - "Убедитесь, что, если это применимо, установлены параметры безопасности SELinux (Автоматизировано)."**

<img width="606" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/c2e96a4f-9d92-4903-b711-4b083d83daf9">

Степень риска: Отсутствие установленных параметров безопасности SELinux для контейнеров (в данном случае для сервера и PostgreSQL) может повлиять на безопасность контейнеров, поскольку SELinux предоставляет механизмы обеспечения безопасности на уровне ядра Linux.

Как исправить:
1. Установка параметров SELinux: Настройте и примените соответствующие параметры безопасности SELinux для контейнеров для усиления защиты и контроля доступа.
2. Тестирование и проверка: Проведите проверки и тестирование параметров SELinux для обеспечения их правильной работы без сбоев.

**5.8 - Ensure privileged ports are not mapped within containers (Automated) - "Убедитесь, что привилегированные порты не мапятся внутри контейнеров (Автоматизировано)."**

<img width="574" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/74f28afb-b26e-4da1-b6bc-8f010b7d15c0">

Степень риска: Использование привилегированных портов в контейнерах (например, порт 80) может увеличить риск безопасности, поскольку это дает контейнеру более широкие права доступа, чем требуется, что потенциально может привести к уязвимостям и атакам.

Как исправить:
1. Изменение портов: Измените использование привилегированных портов на непривилегированные для контейнеров, если это возможно.
2. Использование нестандартных портов: Переназначьте приложения на нестандартные порты (>1024), оставляя привилегированные порты для обратного прокси или балансировщика нагрузки.

**5.9 - Ensure that only needed ports are open on the container (Manual) - "Убедитесь, что открыты только необходимые порты в контейнере (ручной режим)."**

<img width="551" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/fa1e1669-47fe-48d2-9ecd-3714a28e74b0">

Степень риска: Открытые ненужные порты в контейнере (например, порт 80) могут увеличить атаку на контейнер и внутренние сервисы, увеличивая поверхность атаки и риск компрометации.

Как исправить:
1. Закрытие ненужных портов: Закройте порты, которые не используются в контейнере, такие как порт 80, если он не требуется конкретным сервисом.
2. Мониторинг открытых портов: Убедитесь, что только необходимые порты открыты и мониторятся для обнаружения внезапных или несанкционированных открытий портов.

**5.11 - Ensure that the memory usage for containers is limited (Automated) - "Убедитесь, что для контейнеров установлено ограничение на использование памяти (Автоматизировано)."**

<img width="569" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/f99bd3e5-ba5a-46b1-9269-4174eed56089">

Степень риска: Запуск контейнеров без установленного ограничения на использование памяти может привести к излишнему расходу ресурсов, конфликтам и нестабильности системы. Это также может привести к перегрузке системы и падениям контейнеров.

Как исправить:
1. Установка ограничений памяти: Укажите максимальное использование памяти для контейнеров с помощью параметров --memory или --memory-swap при запуске контейнера.
2. Мониторинг использования ресурсов: Следите за использованием памяти контейнеров и подстраивайте установленные ограничения в соответствии с потребностями приложения.

**5.12 - Ensure that CPU priority is set appropriately on containers (Automated) - "Убедитесь, что приоритет ЦПУ на контейнерах установлен соответствующим образом (Автоматизировано)."**

<img width="603" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/97071933-cb46-44e6-bc3d-943c3742bc7c">

Степень риска: Запуск контейнеров без установленного приоритета CPU (ограничений) может привести к нежелательной конкуренции за ресурсы процессора, перегрузке сервера и нестабильной работе приложений. Это также может привести к непредвиденным простоям или сбоям в работе контейнеров.

Как исправить:
1. Установка ограничений CPU: Укажите максимальное содержание CPU на контейнер с помощью параметров --cpus при запуске контейнера.
2. Мониторинг использования ресурсов: Проводите мониторинг использования CPU контейнерами и приспосабливайте установленные ограничения в соответствии с требованиями приложений.

**5.13 - Ensure that the container's root filesystem is mounted as read only (Automated) - "Убедитесь, что корневая файловая система контейнера смонтирована только для чтения (Автоматизировано)."**

<img width="654" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/b8b7ff80-fde4-4a54-8dd6-4e1f9eb63a79">

Степень риска: Запуск контейнера с корневой файловой системой, смонтированной для чтения и записи, увеличивает риск изменения или повреждения системных файлов, что может привести к утечке информации и нарушению работоспособности приложений.

Как исправить:
1. Настройка корневой файловой системы: Установите режим только для чтения для корневой файловой системы контейнера.
2. Использование volume mounts: Используйте volume mounts для сохранения данных, а не изменения корневой файловой системы контейнера.


**5.14 - Ensure that incoming container traffic is bound to a specific host interface (Automated) - "Убедитесь, что входящий сетевой трафик контейнера привязан к конкретному сетевому интерфейсу хоста (Автоматизировано)."**

<img width="720" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/2ff5fa4c-3dbc-4822-96d1-371a0ec4ec16">

Степень риска: Привязка портов контейнера к широко используемому IP-адресу (0.0.0.0) может увеличить риск нарушений безопасности и расширить поверхность атак, из-за возможности обращения к портам с любого IP-адреса.

Как исправить:
1. Ограничение привязки портов: Привяжите входящий сетевой трафик контейнера к конкретному IP-адресу хоста, а не к 0.0.0.0, чтобы уменьшить уязвимости.
2. Использование брандмауэра: Настройте брандмауэр, чтобы допускать трафик только через необходимые порты и IP-адреса для улучшения безопасности.

**5.26 - Ensure that the container is restricted from acquiring additional privileges (Automated) - "Убедитесь, что контейнер ограничен в приобретении дополнительных привилегий (Автоматизировано)."**

<img width="734" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/c3541338-da92-4b8d-8537-5a32967dd1fe">

Степень риска: Если контейнерам разрешено приобретать дополнительные привилегии, это может стать потенциальной точкой входа для злоумышленников, увеличив риск нежелательной доступности и контроля над системой.

Как исправить:
1. Установка ограничений привилегий: Убедитесь, что у контейнера установлены ограничения привилегий, чтобы ограничить его возможности приобретения дополнительных привилегий.
2. Мониторинг безопасности: Регулярно мониторьте привилегии контейнера и реагируйте на любые попытки приобретения нежелательных привилегий.

**5.29 - Ensure that the PIDs cgroup limit is used (Automated) - "Убедитесь, что установлено ограничение на количество процессов (PIDs cgroup limit) (Автоматизировано)."**

<img width="479" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/99c83d6d-794a-432f-8b74-fe640da7f1e1">

Степень риска: Отсутствие установленного ограничения на количество процессов (PIDs) в контейнерах (например, для сервера и PostgreSQL) может привести к перегрузке системы, утечкам ресурсов и возможным атакам на отказ в обслуживании.

Как исправить:
1. Установка лимита PIDs: Укажите максимальное количество процессов, которые может запускать контейнер, с помощью параметра --pids-limit при запуске контейнера.
2. Мониторинг процессов: Отслеживайте количество процессов в контейнерах и адаптируйте установленные лимиты в соответствии с потребностями приложений.

# docker-scout
Проверим уязвимости с помощью docker-scout. Для этого выберем образ и выполним анализ
В результате будут обнаружены те же самые уязвимости, что выявила утилита trivy

<img width="1440" alt="image" src="https://github.com/irefuse3585/docker-vulnerabilities/assets/76556696/a04df30c-9e35-440b-9e48-ab3d28438a89">
