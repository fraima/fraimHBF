**Выбор инструмента**
============

Изначально мы решили использовать ebpf, так как он имеет ряд явных преимуществ перед nftables и iptables. Однако, учитывая тот факт, что крупные FinTech компании все еще используют устаревшие версии ядра Linux ниже 4.14, мы были вынуждены пересмотреть наше решение в пользу Nftables (о преимуществах ниже).

Тем не менее, мы всегда учитываем потребности наших клиентов и разрабатываем архитектуру с учетом возможности использования FraimHBF на ebpf. Мы уверены, что в дальнейшем это позволит обеспечить максимальную эффективность и безопасность работы нашей инфраструктуры для всех наших пользователей.

Iptables
------------
???+Плюсы tip
    - Широко распространен и имеет долгую историю использования.
    - Обладает хорошей документацией и множеством ресурсов для обучения и отладки.
    - Мощный инструмент для управления трафиком с большим количеством опций и возможностей.
    - Хорошо поддерживается и имеет широкое сообщество пользователей.
    - Может быть использован вместе с другими инструментами, такими как fail2ban, для дополнительной защиты сервера.

???+Минусы danger
    - Имеет сложный и запутанный синтаксис, что может быть проблематично для начинающих пользователей.
    - Не обеспечивает полностью декларативную конфигурацию, что может сделать отслеживание и управление правилами более сложным.
    - Некоторые функции, такие как обработка фрагментов пакетов, могут работать медленнее, чем в nftables.
    - Не поддерживает все возможности, которые доступны в nftables.

eBPF
---------
???+Плюсы tip
    - Одним из главных преимуществ eBPF является высокая производительность и гибкость, которые достигаются за счет использования JIT-компиляции и возможности динамической загрузки и отключения программ. eBPF также обладает более широким набором инструментов для анализа сетевых пакетов и принятия решений на основе различных метрик, что может быть полезным в условиях высокой нагрузки и сложной сетевой инфраструктуры.
    - Гибкость: eBPF является универсальным инструментом, который может использоваться не только в качестве брандмауэра, но и для других задач, таких как мониторинг и анализ сетевого трафика.
    - Простота в использовании: eBPF имеет простой и понятный синтаксис, что делает его доступным для широкого круга пользователей.

???+Минусы danger
    - Ограничения в возможностях: eBPF не поддерживает некоторые функции, которые могут быть полезны в качестве брандмауэра, такие как отслеживание состояния соединения.
    - Сложность настройки: eBPF требует определенного уровня знаний, что может сделать его настройку и использование более сложным для неопытных пользователей.

Nftables
---------
???+Плюсы tip 
    - Обладает более чистым и простым синтаксисом, чем iptables, что упрощает конфигурирование и отладку правил.
    - Предоставляет полностью декларативный подход к конфигурации брандмауэра.
    - Обрабатывает фрагменты пакетов быстрее, чем iptables.
    - Поддерживает множество функций, таких как отложенное применение правил и автоматическое управление временными правилами.
    - Есть поддержка динамического обновления правил без необходимости перезагрузки или пересборки целевого ядра. Это особенно полезно в случаях, когда требуется быстрое изменение конфигурации брандмауэра в реальном времени.
    - Есть поддержка групповых правил для более эффективного управления правилами, связанными с различными группами пакетов.
    - Есть поддержка более сложных типов матчей, которые могут использоваться для определения конкретных условий, таких как наличие конкретных флагов TCP или определенных значений IP-адресов.
    - Есть поддержка нативного синтаксиса для работы с IP-адресами и другими типами данных, что делает конфигурацию брандмауэра более простой и удобной.
    - Есть поддержка отслеживания состояния соединений, что позволяет создавать более сложные правила, основанные на состоянии соединения.
???+Минусы danger
    - Относительно новый инструмент, и имеет меньшее сообщество пользователей и меньшее количество ресурсов для обучения и отладки, чем iptables.
    - Некоторые функции, такие как поддержка определенных модулей ядра, могут быть ограничены.
    - Не обладает всеми возможностями, которые доступны в iptables.

