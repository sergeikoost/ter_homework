# Задача 1

В прошлой домашней работе установил все необходимое для выполнения этой.

Работу выполнял прямо из файлов демонстрационной лекции немного изменяя или дополняя их.

Немного изменяем файл cloud-init.yml

```
#cloud-config
users:
  - name: ubuntu
    groups: sudo
    shell: /bin/bash
    sudo: ["ALL=(ALL) NOPASSWD:ALL"]
    ssh_authorized_keys:
      - ${ssh_key}
package_update: true
package_upgrade: false
packages:
  - vim
  - nginx
```

Тут мы использовали переменную для ssh-ключа вместо хардкода, также добавили package nginx для дальнейшего выполнения задачи.

Следующим шагом я объявил все переменные:

```
###cloud vars

variable "ssh_key" {
  type    = string
  default = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPzPE5D2fUw+N2HRvttozFcURLdaxZ5SFz9EkxKDRUzS root@ubuntulearn"
}
variable "token" {
  type        = string
  description = "Yandex Cloud token"
}

variable "cloud_id" {
  type        = string
  description = "Yandex Cloud ID"
}

variable "folder_id" {
  type        = string
  description = "Yandex Cloud Folder ID"
}
```


Переменные token, cloud_id и folder_id создал/изменил в personal.auto.tfvars прописав свои данные. 

Изменил файл main.tf, теперь выглядит так:

```
# Создаем сеть и подсети
resource "yandex_vpc_network" "develop" {
  name = "develop"
}

resource "yandex_vpc_subnet" "develop_a" {
  name           = "develop-ru-central1-a"
  zone           = "ru-central1-a"
  network_id     = yandex_vpc_network.develop.id
  v4_cidr_blocks = ["10.0.1.0/24"]
}

resource "yandex_vpc_subnet" "develop_b" {
  name           = "develop-ru-central1-b"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.develop.id
  v4_cidr_blocks = ["10.0.2.0/24"]
}

data "template_file" "cloudinit" {
  template = file("./cloud-init.yml")
  vars = {
    ssh_key = var.ssh_key  # Используем переменную ssh_key
  }
}

# ВМ для проекта marketing
module "marketing_vm" {
  source         = "git::https://github.com/udjin10/yandex_compute_instance.git?ref=main"
  env_name       = "marketing"
  network_id     = yandex_vpc_network.develop.id
  subnet_zones   = ["ru-central1-a"]
  subnet_ids     = [yandex_vpc_subnet.develop_a.id]
  instance_name  = "marketing-vm"
  instance_count = 1
  image_family   = "ubuntu-2004-lts"
  public_ip      = true

  metadata = {
    user-data          = data.template_file.cloudinit.rendered
    serial-port-enable = 1
  }

  labels = {
    owner   = "marketing-team"
    project = "marketing"
  }
}

# ВМ для проекта analytics
module "analytics_vm" {
  source         = "git::https://github.com/udjin10/yandex_compute_instance.git?ref=main"
  env_name       = "analytics"
  network_id     = yandex_vpc_network.develop.id
  subnet_zones   = ["ru-central1-b"]
  subnet_ids     = [yandex_vpc_subnet.develop_b.id]
  instance_name  = "analytics-vm"
  instance_count = 1
  image_family   = "ubuntu-2004-lts"
  public_ip      = true

  metadata = {
    user-data          = data.template_file.cloudinit.rendered
    serial-port-enable = 1
  }

  labels = {
    owner   = "analytics-team"
    project = "analytics"
  }
}
```

Проверяем terraform plan, все выглядит рабочим, применяем terraform plan:

![terraform_homework3-task1 1](https://github.com/user-attachments/assets/8e20e1d4-76c8-4588-830c-a031c2d2bd4a)


Создалось 2 вм:

![terraform_homework3-task1 2](https://github.com/user-attachments/assets/4e7d432a-8f38-40a8-8fa0-71cb3dc3067a)



Заходим по очереди на виртуалки с машины, на которой выполнялась работа и даем команду sudo nginx -t

1-ая вм:


![terraform_homework3-task1 3](https://github.com/user-attachments/assets/2c9e7a17-1df2-44d5-8513-0c1f98cf0d59)


2-ая вм:

![terraform_homework3-task1 4](https://github.com/user-attachments/assets/5b23e642-b986-475e-87e7-27785d2edda4)


# Задача 2

Создаем модуль vpc в отдельной директории modules/vpc/

В нем делаю 4 файла, которые будут использоваться в этом модуле.

main.tf - тут мы создадем ресурсы сети и подсети:

```
resource "yandex_vpc_network" "network" {
  name = var.env_name
}

resource "yandex_vpc_subnet" "subnet" {
  name           = "${var.env_name}-${var.zone}"
  zone           = var.zone
  network_id     = yandex_vpc_network.network.id
  v4_cidr_blocks = [var.cidr]
}
```

outputs.tf - тут определяем выходные значения для модуля:

```
output "network_id" {
  value = yandex_vpc_network.network.id
}

output "subnet_id" {
  value = yandex_vpc_subnet.subnet.id
}

output "subnet_info" {
  value = yandex_vpc_subnet.subnet
}
```


providers.tf - тут указываем терраформ провайдера: 

```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
}
```


variables.tf - тут определяем входные переменные для модуля:

```
variable "env_name" {
  type        = string
  description = "Environment name (e.g., develop, stage, prod)"
}

variable "zone" {
  type        = string
  description = "Availability zone for the subnet"
}

variable "cidr" {
  type        = string
  description = "CIDR block for the subnet"
}
```


Далее необходимо подключить этот модуль к основному проекту, сделаем это немного исправив файл main.tf в основном проекте:

```
# Создаем сеть и подсеть с помощью локального модуля vpc
module "vpc_dev" {
  source   = "./modules/vpc"  # Указываем путь к локальному модулю
  env_name = "develop"        # Название среды
  zone     = "ru-central1-a"  # Зона доступности
  cidr     = "10.0.1.0/24"    # CIDR блок для подсети
}
```

В файле outputs.tf добавим вывод информации о модуле vpc:

```
output "vpc_dev_info" {
  value = module.vpc_dev.subnet_info
}
```


Применяем конфигурацию terraform apply, проект собирается без проблем, прикладываю output:

![terraform_homework3-task2 1](https://github.com/user-attachments/assets/c7a9f965-21b9-4ac0-801d-be0e97f37326)


Устанавливаем terraform-docs и формируем документацию для нашего модуля vpc командой:


```
terraform-docs markdown ./modules/vpc > ./modules/vpc/README.md
```


Создается документация, прикладываю скрин, но если нужно текстом могу и текстом:

![terraform_homework3-task2 2](https://github.com/user-attachments/assets/689bd157-16be-4dd1-a15e-b0b45dbe1bf4)


Заходим в консоль терраформа и просим показать вывод информации о модуле, конкретнее о subnet, информация корректная:

![terraform_homework3-task2 3](https://github.com/user-attachments/assets/24f77c92-f30f-47f8-86a2-6b8772d9b441)

# Задание 3

1. Выводим список ресурсов в стейте, terraform state list:

![terraform_homework3-task3 1](https://github.com/user-attachments/assets/eeac1d5f-5c1f-4879-bf05-689ab0bf2991)


2. Удаляем из стейта модуль vpc:

![terraform_homework3-task3 2](https://github.com/user-attachments/assets/ac71bc61-4c18-4148-a039-31859060e7f9)

3. Удаляем из стейта модуль vm:

![terraform_homework3-task3 3](https://github.com/user-attachments/assets/a00f56c6-b099-4d0c-b981-de8c43da3923)

4. Возвращаем:

Первым делом вернем сети, предварительно узнав их ID в клауд консоли, команды:

```
terraform import module.vpc_dev.yandex_vpc_network.network enpml8es7vo280kqggia
terraform import module.vpc_dev.yandex_vpc_subnet.subnet e9b2ri9ps4fa3ol1vhh6
```

Дальше уже по очереди вернем модуль vm в стейт для каждой нашей vm, id также берем из клауда:


```
terraform import module.marketing_vm.yandex_compute_instance.vm[0] fhmop8i2itijn1m3tke5
terraform import module.analytics_vm.yandex_compute_instance.vm[0] fhmapje6diqkrbodsdhg
```


Вывод terraform plan:

![terraform_homework3-task3 4](https://github.com/user-attachments/assets/62c6d393-e520-408e-98d6-c50eff709d87)


Значимых изменений нет.
