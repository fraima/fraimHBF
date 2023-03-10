
** Структура: **
========================

???+ failure 
    - Те кто работают с K8S кластерами, видили как раздувается iptables в кол-ве правил из-за чего
    время обновления правил могли достигать минуты.

Т.к мы больше не обременены ограничениями iptables, мы можем создать отдельную таблицу и без опасения задеть чей-то chain, формируем требуемые структуры.

Кол-во правил мы смогли сократить благодаря простому подходу 5 tuple: 
>`srcIP`, `srcPORT`, `dstIP`, `dstPORT`, `protocol`  | verdict

где `srcIP`, `srcPORT`, `dstIP`, `dstPORT` являются IPSET массивами с rangeIP, rangePort.

Для группировки мы вводим такое понятие как SG=Security-Group.

???+ info
    SG - это своего рода логическая группа, которая включает в себя список подсетей от /32 до /0
    и обладает двумя атрибутами:

    - Owner
    - Subnets

???+ danger
    Главное условие - одна подсеть может находиться только в одной SG.
    
    >success:
    {"sg-1":
        "10.0.0.0/25"
    "sg-2":
        "10.0.0.128/25"}

    >failed:
    {"sg-1":
        "10.0.0.0/24"
    "sg-2":
        "10.0.0.128/25"}
    
    как видим 10.0.0.128/25 является частью подсети 10.0.0.0/24 

SG мы можем наполнять как нам угодно, в рамках какого-то процесса.
Например при создании виртуальной машины в cloud или onPrem, как только ей выделили IP адрес, мы можем добавить его в 
``SG-1 append 1.1.1.1/32 ``

???+ info
    - очередность ip адресов не важна, в рамках одной SG могут находиться подсети из разных подсетей (отдается на откуп ИБ и администраторов)
    - система будет стараться объединять соседние подсети в подсети высшего порядка, если есть такая возможность.

Таким образом у нас может появиться например две SG (SG-TEAM-A, SG-TEAM-B) с примером:

>Нужно организовать правило по которому 
SG-TEAM-A может ходить в SG-TEAM-B по порту 80,443

Для такого запроса, в nftables веделится 4 IPSET массива:
>`srcIP`, `srcPORT`, `dstIP`, `dstPORT`
которые заполнятся RangeIP, RangePorts согласно правилу.

Дальше остается только сформировать правило на (SG-TEAM-A, SG-TEAM-B) узлах.

Для SG-TEAM-A будет сформирована цепочка OUTPUT c правилом

`от SG-TEAM-A:IP SG-TEAM-A:PORT до SG-TEAM-B:IP SG-TEAM-B:PORT | protocol | accept`

Для SG-TEAM-B будет сформирована цепочка INPUT c правилом

`от SG-TEAM-A:IP SG-TEAM-A:PORT до SG-TEAM-B:IP SG-TEAM-B:PORT | protocol | accept`

Server
------
Сервер является ядром системы и позволяет клиентам забирать свою актуальную схему сетевых политик. 

Client
------
Клиент настраивает firewall узла, основываясь на IP адресах указанные на его интерфейсах и ответе от API,
в котором передается актуальный список правил, для каждой цепочки `INPUT`, `OUTPUT`, `FORWARD`.

``` mermaid
sequenceDiagram
    autonumber
    Client->>Netlink: есть че?
    Netlink->>Client: держи IP!
    loop Healthcheck
        Netlink->>Client: что-то обновилось
    end
    Client->>API: Кто я?
    API->>Client: Ты тварь дрожащая.
    Client->>API: А какие у меня права?
    API->>Client: Умеешь общаться с  TEAM-A
    Client->>API: Кто такие TEAM-A?
    API->>Client: Это четки пацаны!
    Client->>Nftables: Ну ок добавлю правило


```
