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
