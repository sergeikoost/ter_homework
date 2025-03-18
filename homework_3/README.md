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
