Изучите проект. В файле variables.tf объявлены переменные для Yandex provider.
Создайте сервисный аккаунт и ключ. service_account_key_file.
Сгенерируйте новый или используйте свой текущий ssh-ключ. Запишите его открытую(public) часть в переменную vms_ssh_public_root_key.

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

yc resource-manager folder add-access-binding b1gjbscn2p28l9st4t7e --role editor --subject serviceAccount:ajec7nr43f2ee9108mnf
