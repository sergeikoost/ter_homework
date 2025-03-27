# Задача 1

Изучить проект, запустить и показать вывод  входящих правил «Группы безопасности»


Логика работы:


1 - Terraform инициализирует провайдер Yandex Cloud с указанными учетными данными

2 - Создается VPC сеть с заданным именем

3 - В этой сети создается подсеть в указанной зоне с заданным CIDR-блоком

4 - Настраиваются правила входящего трафика для SSH, HTTP и HTTPS

Инициализировал проект, все прошло успешно, вывод групп ниже:

![terraform_homework4-task1 1](https://github.com/user-attachments/assets/8f555cda-ac67-459e-ad74-71258a4f5ef7)
![terraform_homework4-task1 2](https://github.com/user-attachments/assets/210a065a-49ff-41d8-a12c-13b6522db5a1)


# Задача 2


1 - Создайте файл count-vm.tf. Опишите в нём создание двух одинаковых ВМ web-1 и web-2 (не web-0 и web-1) с минимальными параметрами, используя мета-аргумент count loop. Назначьте ВМ созданную в первом задании группу безопасности.(как это сделать узнайте в документации провайдера yandex/compute_instance )


Файл count-vm.tf:

```
# count-vm.tf
resource "yandex_compute_instance" "web" {
  count = 2 # Используем count = 2 для создания двух идентичных ВМ
  name = "web-${count.index + 1}" # Имена формируются как web-1 и web-2 через ${count.index + 1}

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd8vmcue7aajpmeo39kk" # Ubuntu 20.04 LTS
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.develop.id
    nat       = true
    security_group_ids = [yandex_vpc_security_group.example.id] # Указываем группу безопасности
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_ed25519.pub")}"
  }

  depends_on = [yandex_compute_instance.db] # Указывает зависимость от ВМ БД через depends_on
}
```

2 - Создайте файл for_each-vm.tf. Опишите в нём создание двух ВМ для баз данных с именами "main" и "replica" разных по cpu/ram/disk_volume , используя мета-аргумент for_each loop. Используйте для обеих ВМ одну общую переменную типа

```
# for_each-vm.tf
locals {
  ssh_key = file("~/.ssh/id_ed25519.pub") #Используем local-переменную для чтения SSH-ключа
}

resource "yandex_compute_instance" "db" {
  for_each = { for vm in var.each_vm : vm.vm_name => vm } #Используем for_each с преобразованием списка в map

  name = each.key # "main" или "replica"

  resources {
    cores  = each.value.cpu
    memory = each.value.ram
  }

  boot_disk {
    initialize_params {
      image_id = "fd8vmcue7aajpmeo39kk" # Ubuntu 20.04 LTS
      size     = each.value.disk_volume
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.develop.id
    nat       = true
    security_group_ids = [yandex_vpc_security_group.example.id] # Группа безопасности
  }

  metadata = {
    ssh-keys = "ubuntu:${local.ssh_key}"
  }
}
```


Параметры берутся из переменной each_vm, код ниже:

```
variable "each_vm" {
  type = list(object({
    vm_name     = string
    cpu         = number
    ram         = number
    disk_volume = number
  }))
  default = [
    {
      vm_name     = "main"
      cpu         = 4
      ram         = 4
      disk_volume = 20
    },
    {
      vm_name     = "replica"
      cpu         = 2
      ram         = 2
      disk_volume = 10
    }
  ]
}
```

Запускаем проект. Создались 4 вм:

![terraform_homework4-task2 1](https://github.com/user-attachments/assets/a430621e-8207-4819-ad4f-d4b3bd6febe2)


Все созданные ВМ будут использовать указанную подсеть, иметь публичный IP, иметь доступ по SSH с моим ключом и подчиняться правилам, которые описаны в файле security.tf


# Задача 3

Создаем файл disk_vm.tf:

```
resource "yandex_compute_disk" "additional_disks" {
  count = 3 #Создаем 3 одинаковых диска
  name  = "disk-${count.index + 1}" #Присваиваем имена дискам
  size  = 1 # Задаем размер дисков
  zone  = var.default_zone
}

# Создаем одиночную ВМ "storage" с подключением дисков
resource "yandex_compute_instance" "storage" {
  name = "storage"

  resources {
    cores  = 2
    memory = 2
  }

  boot_disk {
    initialize_params {
      image_id = "fd8vmcue7aajpmeo39kk" # Ubuntu 20.04 LTS
      size     = 10
    }
  }

  network_interface {
    subnet_id = yandex_vpc_subnet.develop.id
    nat       = true
    security_group_ids = [yandex_vpc_security_group.example.id]
  }

  metadata = {
    ssh-keys = "ubuntu:${file("~/.ssh/id_ed25519.pub")}"
  }

  # Динамическое подключение дополнительных дисков
  dynamic "secondary_disk" {             
    for_each = { for idx, disk in yandex_compute_disk.additional_disks : idx => disk.id }#Динамический блок создает секцию secondary_disk для каждого диска
    content {
      disk_id = secondary_disk.value #disk_id берется из созданных дисков (yandex_compute_disk.additional_disks.*.id)
    }
  }
}

```
Итого создалось 5 вм с указанными ранее параметрами, 4 из прошлой задачи и 1 новая, storage

![terraform_homework4-task3 1](https://github.com/user-attachments/assets/7132cb99-aa23-4901-8ec3-fa594116663f)



Поскольку нас интересует именно storage вм и создание 3-ех дисков на ней прикладываю скрин с этой вм:

![terraform_homework4-task3 3](https://github.com/user-attachments/assets/7a9c35f0-059d-4afe-af87-20970b4838f8)


#Задача 4

