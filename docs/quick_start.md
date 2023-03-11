
**Как начать?**
---------------

На данный момент мы предлагаем два способа настройки нашей системы: 

- Напрямую через использование API. -> swagger
- С помощью Terraform провайдера.

Если вы выберете настройку через API, то вам необходимо будет создать запросы к нашему API, чтобы интегрировать систему в ваш процесс.

Альтернативно, если вы выберете настройку через Terraform, то вам нужно будет воспользоваться нашим провайдером и определить конфигурацию системы в Terraform файле. Это позволит вам быстро и легко настроить систему с помощью готовых инструментов.

Независимо от выбранного способа настройки, мы готовы помочь вам достичь требуемого результата и интегрировать FraimHBF в вашу инфраструктуру.


**Пример**
--------
Предположим, что у нас две команды `teamA` и `teamB`, команда `teamA` пишет бекенд, а `teamB` пишет фронтенд.
Требуется создать две виртуальные машины одну для `teamA` вторую для `teamB` и открыть доступ от `teamA/backend` до `teamB/frontend` по `80/TCP`.

Для реализации данной задачи мы ввели 4 абстракции.


- `unit` - владелец области (`teamA`, `teamB`)
- `security group` - виртуальная группа области владельца в которой находятся подсети, логически сгруппированных узлов. (`frontend`, `backend`)
- `networks` - подсети управляемых узлов.
- `rule` - правила доступа между `security group` как в рамках одного так и в рамках разных `unit`.

=== "Yandex-Cloud"

    ``` { .tf }
        <настройки провайдера>

        # Определяем базовую виртуальную сеть, в которой будут жить ВМ.
        resource "yandex_vpc_network" "network-1" {
            name = "network1"
        }

        # Определяем базовую подсеть из которой будут выдаваться адреса для ВМ.
        resource "yandex_vpc_subnet" "subnet-1" {
            name           = "subnet1"
            zone           = "ru-central1-a"
            network_id     = yandex_vpc_network.network-1.id
            v4_cidr_blocks = ["192.168.10.0/24"]
        }

        # Создает ВМ для teamA/backend
        resource "yandex_compute_instance" "teamA_backend" {
            name = "teamA_backend"

            resources {
                cores  = 2
                memory = 2
            }

            boot_disk {
                initialize_params {
                image_id = "fd87va5cc00gaq2f5qfb"
                }
            }

            network_interface {
                subnet_id = yandex_vpc_subnet.subnet-1.id
                nat       = true
            }

            metadata = {
                ssh-keys = "ubuntu:${file("~/.ssh/id_ed25519.pub")}"
            }
        }

        # Создает ВМ для teamB/frontend
        resource "yandex_compute_instance" "teamB_frontend" {
            name = "teamB_frontend"

            resources {
                cores  = 4
                memory = 4
            }

            boot_disk {
                initialize_params {
                image_id = "fd87va5cc00gaq2f5qfb"
                }
            }

            network_interface {
                subnet_id = yandex_vpc_subnet.subnet-1.id
                nat       = true
            }

            metadata = {
                ssh-keys = "ubuntu:${file("~/.ssh/id_ed25519.pub")}"
            }
        }

        # Получает локальный адрес ВМ
        output "internal_ip_address_teamA_backend" {
            value = yandex_compute_instance.teamA_backend.network_interface.0.ip_address
        }

        # Получает локальный адрес ВМ
        output "internal_ip_address_teamB_frontend" {
            value = yandex_compute_instance.teamB_frontend.network_interface.0.ip_address
        }

    ```

=== "FraimHBF"

    ```{ .tf .annotate }
        <настройки провайдера> 

        locals {
            # Заранее вычисляем список подсетей для teamA_backend
            teamA_backend_list  = [
                "${yandex_compute_instance.teamA_backend.network_interface.0.ip_address}/32"
            ]   
            # Заранее вычисляем список подсетей для teamB_frontend
            teamB_frontend_list = [
                "${yandex_compute_instance.teamB_frontend.network_interface.0.ip_address}/32"
            ]

            # Описываем какие доступы требуется открыть
            rules = [
                {
                    description = "Access from backend to frontend"
                    proto       = "tcp"
                    sg_from     = "teamA_backend"
                    sg_to       = "teamB_frontend"
                    ports_to    = "80"
                }
            ]

            # Описываем какие сети у нас есть и их содержимое
            networks = [
                {
                    name    = "teamA_backend"
                    cidrs   = local.teamA_backend_list
                },
                {
                    name    = "teamB_frontend"
                    cidrs   = local.teamB_frontend_list
                }

            ]

            # Описываем группы безопасности и подключенные к ней подсети
            security_groups = [
                {
                    name        = "teamA_backend",
                    networks    = "teamA_backend"
                },
                {
                    name        = "teamB_frontend",
                    networks    = "teamB_frontend"
                },
            ]

            # Конвертируем список в словарь, для использования в for_each
            networks_map = { for item in local.networks :
                keys(item)[0] => values(item)[0]
            }
            rules_map = { for item in local.rules :
                keys(item)[0] => values(item)[0]
            }
            security_groups_map = { for item in local.security_groups :
                keys(item)[0] => values(item)[0]
            }
        }

        # Создает HBF сети в которых хранятся подсети заказанных ВМ.
        resource "sgroups_networks" "nws" {
            for_each = local.networks_map

            name     = each.value.name
            cidrs    = each.value.cidrs
        }

        # Создает группы безопасности
        resource "sgroups_groups" "groups" {
            depends_on = [
                sgroups_networks.nws
            ]

            for_each    = local.security_groups_map

            name        = each.value.name
            networks    = each.value.networks

        }

        # Создает сетевые политики
        resource "sgroups_rules" "rules" {
            depends_on = [
                sgroups_groups.groups
            ]
            for_each    = local.rules_map

            proto       = each.value.proto
            sg_from     = each.value.sg_from
            sg_to       = each.value.sg_to
            ports_from  = try(each.value.ports_from, "*")
            ports_to    = try(each.value.ports_to,   "*")
        }




    ```