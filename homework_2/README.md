# Задача 1

Для удобства установил yandex cli для работы в терминале:

```
root@ubuntulearn:~# curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh -o install.sh
root@ubuntulearn:~# chmod +x install.sh
root@ubuntulearn:~# ./install.sh
```

Далее yc init, после успешной настройки создаем сервисный аккаунт:

```
yc iam service-account create --name terhomework
```

После чего выполняем настройку:


```
yc resource-manager folder add-access-binding b1gjbscn2p28l9st4t7e --role editor --subject serviceAccount:ajec7nr43f2ee9108mnf
```
![terraform_homework2-task1 1](https://github.com/user-attachments/assets/cc3afd82-aed5-4320-8b8c-db6e3dbdd400)

Исправляем ошибки в main.tf

1-ая ошибка: в блоке resource "yandex_compute_instance" "platform" написано platform_id = "standart-v4" , надо "standard-v4", просто неверно написано слово.


2-ая ошибка: ключ serial-port-enable должен быть строкой, а не числом, исправляем 

```
metadata = {
    serial-port-enable = "1"
```

3-ая ошибка: 

```
resource "yandex_compute_instance" "platform" {
  name        = "netology-develop-platform-web"
  platform_id = "standard-v4"
```

В официальной документации yandex.cloud нет пплатформы v4, исправляем на v3. Также исправляем аргумент core_fraction и сore потому что  "the specified core fraction is not available on platform "standard-v3"; allowed core fractions: 20, 50, 100", делаем 20 core_fraction и 2 core.


Инициализируем проект со всеми исправлениями:

![terraform_homework2-task1 2](https://github.com/user-attachments/assets/845e4032-9865-4ab2-bc12-5115fb3263dc)


Проверяем что виртуалка создалась, все прошло успешно:

![terraform_homework2-task1 3](https://github.com/user-attachments/assets/21784b72-cefb-4813-8d14-1089389c2560)


Подключаемся к новой машине по ssh и даем команду curl ifconfig.me:

![terraform_homework2-task1 4](https://github.com/user-attachments/assets/276fca10-cace-4777-aafc-f60fb4f676c0)



# Задача 2 

Замените все хардкод-значения для ресурсов yandex_compute_image и yandex_compute_instance на отдельные переменные. К названиям переменных ВМ добавьте в начало префикс vm_web_ . Пример: vm_web_name.


Создаем переменные:

```
variable "vm_web_image_family" {
  type        = string
  default     = "ubuntu-2004-lts"
}

variable "vm_web_name" {
  type        = string
  default     = "netology-develop-platform-web"
}

variable "vm_web_platform_id" {
  type        = string
  default     = "standard-v3"
}

variable "vm_web_cores" {
  type        = number
  default     = 2
}

variable "vm_web_memory" {
  type        = number
  default     = 1
}

variable "vm_web_core_fraction" {
  type        = number
  default     = 20
}

variable "vm_web_preemptible" {
  type        = bool
  default     = true
}

variable "vm_web_nat" {
  type        = bool
  default     = true
}

variable "vm_web_serial_port_enable" {
  type        = string
  default     = "1"
}
```


Обновляем информацию в ресурсах в файле main.tf чтобы он использовал переменные и получаем в итоге такой файл:

```
resource "yandex_vpc_network" "develop" {
  name = var.vpc_name
}
resource "yandex_vpc_subnet" "develop" {
  name           = var.vpc_name
  zone           = var.default_zone
  network_id     = yandex_vpc_network.develop.id
  v4_cidr_blocks = var.default_cidr
}


data "yandex_compute_image" "ubuntu" {
  family = var.vm_web_image_family
}
resource "yandex_compute_instance" "platform" {
  name        = var.vm_web_name
  platform_id = var.vm_web_platform_id
  resources {
    cores         = var.vm_web_cores
    memory        = var.vm_web_memory
    core_fraction = var.vm_web_core_fraction
  }
  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.image_id
    }
  }
  scheduling_policy {
    preemptible = var.vm_web_preemptible
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.develop.id
    nat       = var.vm_web_nat
  }

  metadata = {
    serial-port-enable = var.vm_web_serial_port_enable
    ssh-keys           = "ubuntu:${var.vms_ssh_root_key}"
  }

}
```

terraform plan:

![terraform_homework2-task2 1](https://github.com/user-attachments/assets/2cbf73de-dceb-4104-a23a-5346178292fe)


# Задача 3

Создал файл vms_platform.tf, перенес в него переменные из первой ВМ и сделал переменные для второй, в файле variables.tf закомментировал все переменные с 1-ой ВМ чтобы не было конфликтов:

```
# Переменные для первой виртуалки  (web)

variable "vms_ssh_root_key" {
  type        = string
  default     = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINJi2NOa0VHfHpjpfue1i/mlrbVpr898SDMajPes16gt root@ubuntulearn"
  description = "ssh-keygen -t ed25519"
}
variable "vm_web_image_family" {
  type        = string
  default     = "ubuntu-2004-lts"
}

variable "vm_web_name" {
  type        = string
  default     = "netology-develop-platform-web"
}

variable "vm_web_platform_id" {
  type        = string
  default     = "standard-v3"
}

variable "vm_web_cores" {
  type        = number
  default     = 2
}

variable "vm_web_memory" {
  type        = number
  default     = 1
}

variable "vm_web_core_fraction" {
  type        = number
  default     = 20
}

variable "vm_web_preemptible" {
  type        = bool
  default     = true
}

variable "vm_web_nat" {
  type        = bool
  default     = true
}

variable "vm_web_serial_port_enable" {
  type        = string
  default     = "1"
}

# Переменные для второй VM (db)

#variable "vms_ssh_root_key" {
 # type        = string
  #default     = "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINJi2NOa0VHfHpjpfue1i/mlrbVpr898SDMajPes16gt root@ubuntulearn"
  #description = "ssh-keygen -t ed25519"
#}
variable "vm_db_image_family" {
  type        = string
  default     = "ubuntu-2004-lts"
}

variable "vm_db_name" {
  type        = string
  default     = "netology-develop-platform-db"
}

variable "vm_db_platform_id" {
  type        = string
  default     = "standard-v3"
}

variable "vm_db_cores" {
  type        = number
  default     = 2
}

variable "vm_db_memory" {
  type        = number
  default     = 2
}

variable "vm_db_core_fraction" {
  type        = number
  default     = 20
}

variable "vm_db_preemptible" {
  type        = bool
  default     = true
}

variable "vm_db_nat" {
  type        = bool
  default     = true
}

variable "vm_db_serial_port_enable" {
  type        = string
  default     = "1"
}

variable "vm_db_zone" {
  type        = string
  default     = "ru-central1-b"
}

```


В файл main.tf добавил 2 ресурса для второй ВМ:

1) resource "yandex_vpc_subnet" "develop_b" т.к. указно было сделать что вторая вм должна работать в зоне "ru-central1-b"


```
resource "yandex_vpc_subnet" "develop_b" {
  name           = "${var.vpc_name}-b"
  zone           = "ru-central1-b"
  network_id     = yandex_vpc_network.develop.id
  v4_cidr_blocks = ["10.0.2.0/24"]
}
```

2) resource "yandex_compute_instance" "platform_db" для второй ВМ

```
resource "yandex_compute_instance" "platform_db" {
  name        = var.vm_db_name
  platform_id = var.vm_db_platform_id
  zone        = var.vm_db_zone
  resources {
    cores         = var.vm_db_cores
    memory        = var.vm_db_memory
    core_fraction = var.vm_db_core_fraction
  }
  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.image_id
    }
  }
  scheduling_policy {
    preemptible = var.vm_db_preemptible
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.develop_b.id
    nat       = var.vm_db_nat
  }

  metadata = {
    serial-port-enable = var.vm_db_serial_port_enable
    ssh-keys           = "ubuntu:${var.vms_ssh_root_key}"
  }
}
```

Terraform apply:

![terraform_homework2-task2 2](https://github.com/user-attachments/assets/a15631b7-36d0-43e2-be18-7ebeb5e860f8)

Создалось 2 вм:

![terraform_homework2-task2 3](https://github.com/user-attachments/assets/d17e45ba-676f-43e1-b9d2-e7cfd49c5bf9)


# Задача 4

Объявляю в файле ouputs.tf один output для 2 вм, web и db:

```
output "vm_web_info" {
  description = "Информация о вм с web"
  value = {
    instance_name = yandex_compute_instance.platform.name  # Исправлено на platform
    external_ip   = yandex_compute_instance.platform.network_interface[0].nat_ip_address
    fqdn          = yandex_compute_instance.platform.fqdn
  }
}

output "vm_db_info" {
  description = "Информация о вм с db"
  value = {
    instance_name = yandex_compute_instance.platform_db.name
    external_ip   = yandex_compute_instance.platform_db.network_interface[0].nat_ip_address
    fqdn          = yandex_compute_instance.platform_db.fqdn
  }
}
```

terraform output:


![terraform_homework2-task2 5](https://github.com/user-attachments/assets/78fc14c7-20cc-4c6b-9220-fc061d50e195)


# Задача 5

Файл locals.tf:

```
locals {
  # Имя для первой ВМ (web)
  vm_web_name = "${var.vm_web_name}-${var.default_zone}-${var.vm_web_platform_id}"

  # Имя для второй ВМ (db)
  vm_db_name  = "${var.vm_db_name}-${var.vm_db_zone}-${var.vm_db_platform_id}"
}
```


В файле main.tf изменил 2 строчки чтобы терраформ использовал local переменную:


```
resource "yandex_compute_instance" "platform" {
  name        = local.vm_web_name
resource "yandex_compute_instance" "platform_db" {
  name        = local.vm_db_name  
```
Теперь вывод output формируется на основе переменных в locals.tf

![terraform_homework2-task2 8](https://github.com/user-attachments/assets/621f3ccc-bc0b-41c0-aad5-e6ebcd223a99)


# Задача 6

Объединяем переменные для ресурсов виртуалок в 1-ну переменную:


```
variable "vms_resources" {
  type = map(object({
    cores         = number
    memory        = number
    core_fraction = number
    hdd_size      = number
    hdd_type      = string
  }))
}
```

И общую переменную для метаданных:

```
variable "metadata" {
  type = map(string)
}
```

Создаю отдельный файл terraform.tfvars чтобы внести туда значения перменных, иначе у меня не вышло:


```
vms_resources = {
  web = {
    cores         = 1
    memory        = 2
    core_fraction = 20
    hdd_size      = 10
    hdd_type      = "network-hdd"
  },
  db = {
    cores         = 1
    memory        = 2
    core_fraction = 20
    hdd_size      = 10
    hdd_type      = "network-hdd"
  }
}

metadata = {
  serial-port-enable = "1"
  ssh-keys           = "ubuntu:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAINJi2NOa0VHfHpjpfue1i/mlrbVpr898SDMajPes16gt root@ubuntulearn"
}

```

Также необходимо обновить файл main.tf чтобы терраформ использовал новые переменные:

web вм:

```
resource "yandex_compute_instance" "platform" {
  name        = local.vm_web_name
  platform_id = var.vm_web_platform_id
  zone        = var.default_zone

  resources {
    cores         = var.vms_resources["web"].cores
    memory        = var.vms_resources["web"].memory
    core_fraction = var.vms_resources["web"].core_fraction
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.image_id
      size     = var.vms_resources["web"].hdd_size
      type     = var.vms_resources["web"].hdd_type
    }
  }

  scheduling_policy {
    preemptible = var.vm_web_preemptible
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.develop.id
    nat       = var.vm_web_nat
  }

  metadata = var.metadata
}
```

db вм:

```
resource "yandex_compute_instance" "platform_db" {
  name        = local.vm_db_name
  platform_id = var.vm_db_platform_id
  zone        = var.vm_db_zone

  resources {
    cores         = var.vms_resources["db"].cores
    memory        = var.vms_resources["db"].memory
    core_fraction = var.vms_resources["db"].core_fraction
  }

  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.image_id
      size     = var.vms_resources["db"].hdd_size
      type     = var.vms_resources["db"].hdd_type
    }
  }

  scheduling_policy {
    preemptible = var.vm_db_preemptible
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.develop_b.id
    nat       = var.vm_db_nat
  }

  metadata = var.metadata
}
```

Все неиспользуемые более перменные закомментируем в файле variables.tf.

terraform plan:

![terraform_homework2-task2 9](https://github.com/user-attachments/assets/ceb5e49f-52e6-4657-9ea6-8ce67607c075)
