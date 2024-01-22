University: [ITMO University](https://itmo.ru/ru/) \
Faculty: [FICT](https://fict.itmo.ru) \
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming) \
Year: 2023/2024 \
Group: K34202 \
Author: Shabashov Vasiliy Andreevich \
Lab: Lab3 \
Date of create: 01.12.2023

![1](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/39c8318d-9492-4ca2-a622-f70e9422645c)
![2](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/d37b8a5f-a439-478e-b8e1-560da00cc3e0)
![3](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/cf1e8951-3405-4f4a-89d5-509e8e67ec5b)
![4](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/92a73dfb-9ed9-4c8f-b403-7c6856ff29c9)
![5](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/503e8ec9-99c8-43a7-a931-9b6c56a1cfca)
![6](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/2246fe87-1af7-4783-9e5e-f75cfbaf7f42)
![7](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/efe3091e-bb97-4001-a4ed-2d8be6013fa4)
![8](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/5fa4b6ba-a7d9-499e-ac1d-82fa3544ce09)
![9](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/e709c488-9755-43f7-a8fa-d45b16143e81)
![10](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/772ce48b-17f1-4d88-9fde-a0abc8f62582)
![11](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/62bf9b0b-f8be-4348-8d9a-b72f9d923b98)
![12](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/92065081-06fd-4273-9d9b-1513d35f156f)


# Лабораторная работа №3 "Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"

## Цель работы
С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.

## Ход работы
Для продолжения работы с Netbox мы выполнили установку PostgreSQL. После установки мы создали базу данных netbox и создали в ней пользователя с именем netbox.
```
apt install postgresql
```
```
sudo -u postgres psql
```
```
CREATE DATABASE netbox;
```
```
CREATE USER netbox WITH password 'netbox';
```
```
GRANT ALL PRIVILEGES ON DATABASE netbox to netbox;
```
После этого установили redis командой ```apt install redis-server``` и из [репозитория](https://github.com/netbox-community/netbox/archive/v3.3.9.tar.gz) установили netbox.
```
sudo wget https://github.com/netbox-community/netbox/archive/refs/tags/v3.3.9.tar.gz
```
```
sudo tar -xzf v3.3.9.tar.gz -C /opt
```
```
sudo ln -s /opt/netbox-3.3.9/ /opt/netbox
```
Потом мы создали пользователя
```
groupadd --system netbox
```
```
sudo adduser --system -g netbox netbox
```
```
chown --recursive netbox /opt/netbox/netbox/media/
```
Создали виртуальное окружение, установили зависимости. \
Скопировали и отредактировали файл конфигурации, добавив секретный ключ, сгенерированный командой ```python3 ../generate_secret_key.py```

![config](1.png) \
![config](2.png)

После этого мы запустили миграции, чтобы обновить схему базы данных и применить все необходимые изменения. Потом создали суперпользователя и загрузили статические файлы. \
Затем были установлены и настроены nginx и gunicor. \
Отредактировали файл nginx.conf, был добавлен новый хост и удалены настройки для https сервера.

![nginx](3.png)

После того как мы вошли в систему netbox под учетной записью суперпользователя, которая была предварительно создана, мы добавили информацию о CHR и скачали [файл](netbox_devices.csv).

![CHR](4.png)

Создан файл playbook_chr.yml, в нем название роутера для изменения переносится в файл new_playbook.yml. Информация о роутере берется из файла netbox_devices.csv.

![playbook_chr.yml](5.png)

Успешно выполнили playbook_chr.yml

![playbook_chr.yml](6.png)

Названия CHR1 и CHR2 были изменены с учетом информации из файла new_playbook.yml.

![chr](7.png)

Был сгенерирован токен в Netbox, после чего создан файл playbook_netbox.yml. В данном файле указаны необходимые настройки для внесения изменений в Netbox, включая использование предварительно сгенерированного токена.

![playbook_netbox.yml](8.png)

Успешно выполнили playbook_netbox.yml

![playbook_netbox.yml](9.png)

Зашли в Netbox и увидели новый добавленный девайс

![device](10.png)

## Вывод
С использованием Ansible и Netbox была собрана информация об устройствах, а затем сохранена в отдельном файле. Далее были написаны плейбуки, обеспечивающие связь между Netbox и устройствами CHR. Эти плейбуки выполняют необходимые действия для установки соответствия и обновления данных между Netbox и устройствами CHR.

Пинги

![ping](11.png) \
![ping](12.png)

Схема

![scheme](scheme.png)

Name,Status,Tenant,Site,Location,Rack,Role,Manufacturer,Type,IP Address,ID,Tenant Group,Platform,Serial number,Asset tag,Region,Site Group,Position (U),Rack face,Airflow,IPv4 Address,IPv6 Address,Cluster,Virtual chassis,VC Position,VC Priority,Comments,Contacts,Tags,Created,Last updated
CHR1,Active,,Name,,,Router,Mikrotik,Mikrotik,,1,,,,,,,,,,,,,,,,,,,2023-11-26 13:43,2023-11-26 13:43
CHR2,Active,,Name,,,Router,Mikrotik,Mikrotik,,2,,,,,,,,,,,,,,,,,,,2023-11-26 13:44,2023-11-26 13:44

![image](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/0ee11606-dca4-46f4-952c-f180c09c1387)

![scheme](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/db79a4a8-6835-46da-bf4a-81443b9e78c2)
