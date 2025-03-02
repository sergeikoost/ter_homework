Скачайте и установите Terraform версии >=1.8.4 . Приложите скриншот вывода команды terraform --version.
Скачайте на свой ПК этот git-репозиторий. Исходный код для выполнения задания расположен в директории 01/src.
Убедитесь, что в вашей ОС установлен docker.


Сделал git clone https://github.com/netology-code/ter-homeworks.git, установил на виртуалку docker и terraform.

## Задание 1

1) Перейдите в каталог src. Скачайте все необходимые зависимости, использованные в проекте.


Перешел в каталог ../01/scr/, скопировал файл .terraformrc в домашний каталог и дал команду terraform init, чем инициализировал рабочую директорию для terraform

![terraform_homework1-task1](https://github.com/user-attachments/assets/01688a47-6b69-416e-a76e-63d7b0e60a4c)

2) Изучите файл .gitignore. В каком terraform-файле, согласно этому .gitignore, допустимо сохранить личную, секретную информацию?(логины,пароли,ключи,токены итд)

Файлы с расширением .tfstate и директория .terraform игнориуются в gitignore т.к. они могут содержать чувствительные данные, такие как пароль/логин и прочее. В файле можно сказать явно указано что хранить секреты тут personal.auto.tfvars, и поскольку этот файл добавлен в gitignore он не будет отслеживаться git-ом, значит исключено случайное попадание чувствительных данных в git репозиторий. Также этот файл будет использваться локально, что отлично подходит для хранения секретов. 


3) Выполните код проекта. Найдите в state-файле секретное содержимое созданного ресурса random_password, пришлите в качестве ответа конкретный ключ и его значение.

Даем команду terraform apply:

![terraform_homework1-task1 2](https://github.com/user-attachments/assets/2db1f5d6-4608-47a2-8f0b-e0e4983d0a23)


Ресурс random_password.random_string. Внутри этого ресурса будет ключ result, содержащий сгенерированный пароль:

![terraform_homework1-task1 3](https://github.com/user-attachments/assets/c244c9fe-827e-4f3e-8efa-0916834c59cf)


4) Раскомментируйте блок кода, примерно расположенный на строчках 29–42 файла main.tf. Выполните команду terraform validate. Объясните, в чём заключаются намеренно допущенные ошибки. Исправьте их.

![terraform_homework1-task1 4](https://github.com/user-attachments/assets/cc5d5ecd-ec6b-4d06-b18b-401ce9187f9b)


В целом из ошибки все видно и терраформ явно указывает как её исправить.

В Terraform каждый ресурс должен иметь два идентификатора (метки), тип ресурса (например, docker_image), имя ресурса (уникальное имя, которое вы задаете для этого ресурса). 

Нарушены правила именования, имя ресурса не может начинаться с буквы или спец. символа "_" и должно содержать только буквы, цифры, подчеркивания и дефисы. 

Также допущена ошибка в ссылке на ресурс "name  = "example_${random_password.random_string_FAKE.resulT}"

Имя ресурса random_string_FAKE не соответствует реальному имени ресурса (random_string) и в ключе result T  - большая.


Исправляем в main.tf ошибки и выполняем код:

![terraform_homework1-task1 6](https://github.com/user-attachments/assets/c2fe3c8a-006a-4ec6-bbd3-03b7ed74d403)

![terraform_homework1-task1 5](https://github.com/user-attachments/assets/4e74a09d-6e41-4dc8-839b-002c61f0478f)


В качестве ответа приложите: исправленный фрагмент кода и вывод команды docker ps:

```
resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = true
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "example_${random_password.random_string.result}"

  ports {
    internal = 80
    external = 9090
  }
}
```

![terraform_homework1-task1 7](https://github.com/user-attachments/assets/a95a8ca7-53c3-4a73-a5a1-39ecbff2954a)

5) Замените имя docker-контейнера в блоке кода на hello_world. Не перепутайте имя контейнера и имя образа. Мы всё ещё продолжаем использовать name = "nginx:latest". Выполните команду terraform apply -auto-approve. Объясните своими словами, в чём может быть опасность применения ключа -auto-approve. Догадайтесь или нагуглите зачем может пригодиться данный ключ? В качестве ответа дополнительно приложите вывод команды docker ps.

Меняем строчку в коде:

```
resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "hello_world" # name  = "example_${random_password.random_string.result}" изменен на  "hello_world"
```
Делаем terraform apply -auto-approve:

![terraform_homework1-task1 8](https://github.com/user-attachments/assets/43026e79-81ad-4a3a-8e06-e7045393e319)

![terraform_homework1-task1 9](https://github.com/user-attachments/assets/18a9f4b1-fea4-43ca-88a8-b946db4cfb7b)

И после сносим все что сделали командой terraform destroy -auto-approve:

![terraform_homework1-task1 10](https://github.com/user-attachments/assets/747a94b0-efdc-46e9-8b5b-d511965c66ea)

![terraform_homework1-task1 11](https://github.com/user-attachments/assets/36c461ea-e4fc-4f72-b8ef-60b9d079b116)


docker image не уничтожился из-за строчки в main.tf которая указывает, что образ надо хранить локально и не удалять его после уничтожения ресурсов:

```
resource "docker_image" "nginx" {
  name         = "nginx:latest" 
  keep_locally = true #локально, не удаляем после уничтожения ресурсов terraform
}
```

keep_locally  If true, then the Docker image won't be deleted on destroy operation. If this is false, it will delete the image from the Docker local storage on destroy operation.


# Задание 2


1) Создал хост, установил стек docker, создал пару ключей без пароля для доступа с хостовой машины на удаленную по ssh. Дал доступ к docker daemon-у для удаленного пользователя sudo usermod -aG docker kusk111serj чтобы этот пользователь мог выполнять команды docker cli без sudo.

В документации Terraform провайдера Docker указано, что для подключения к удалённому Docker-хосту через SSH можно использовать параметр host с указанием SSH-соединения.


2) Создал файл main.tf:

```
terraform { #  Terraform загружает провайдеры docker и random
  required_providers { 
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
    random = {
      source  = "hashicorp/random"
      version = "~> 3.5.1"
    }
  }
}

provider "docker" { # Настраиваем подключение к Docker Daemon на удалённой машине через ssh
  host = "ssh://kusk111serj@158.160.71.175:22"
}

provider "random" {} # Инициализирует провайдер random, нужен для генерации случайных чисел, обязательно для задания

resource "random_password" "root_password" { # Генерируем случайные числа для пароля для рута и wordpress юзеров mysql
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}

resource "random_password" "user_password" {
  length      = 16
  special     = false
  min_upper   = 1
  min_lower   = 1
  min_numeric = 1
}

resource "docker_image" "mysql" { # Ресурс docker_image управляет образом, который будет использоваться для создания контейнера, скачиваем mysql образ
  name = "mysql:8"
}

resource "docker_container" "mysql" { # Создаем докер контейнер из образа который скачали выше выполняя инструкцию задачи (порты, окружение и т.д.)
  image = docker_image.mysql.image_id
  name  = "mysql_container"

  env = [
    "MYSQL_ROOT_PASSWORD=${random_password.root_password.result}",
    "MYSQL_DATABASE=wordpress",
    "MYSQL_USER=wordpress",
    "MYSQL_PASSWORD=${random_password.user_password.result}",
    "MYSQL_ROOT_HOST=%"
  ]

  ports {
    internal = 3306
    external = 3306
    ip       = "127.0.0.1"
  }
}
```
3) Создал файл .gitignore чтобы не передать секреты куда не надо:

```
*.tfstate
*.tfstate.*
personal.auto.tfvars
.terraform/
```


4) Подкорректировал файл nano ~/.terraformrc указав provider_installation network_mirror для того чтобы при попытке инициализации не видеть ошибку:

```
Error: Invalid provider registry host
│ 
│ The host "registry.terraform.io" given in provider source address "registry.terraform.io/hashicorp/random" does not offer a Terraform provider registry.
╵
╷
│ Error: Invalid provider registry host
│ 
│ The host "registry.terraform.io" given in provider source address "registry.terraform.io/kreuzwerker/docker" does not offer a Terraform provider registry.

```

5) Даем комманду terraforom init, после terraform aplly:

![tf_homework2_task1 2 apply](https://github.com/user-attachments/assets/deedb47c-3f01-45f0-a878-d9d0313adb27)


6) Видим что все прошло успешно, подключаемся к удаленному хосту по ssh и проверяем что все корректно:

![tf_homework2_task1 2 1](https://github.com/user-attachments/assets/02026ea5-1e4b-4a58-8bbc-5421bbf802eb)


7) Выходим и делаем terraform remove:

![tf_homework2_task1 2 destroy](https://github.com/user-attachments/assets/b14c6e7d-79fe-4ddf-848c-a706b6b6ca05)
