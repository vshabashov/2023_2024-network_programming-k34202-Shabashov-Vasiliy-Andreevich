University: [ITMO University](https://itmo.ru/ru/) \
Faculty: [FICT](https://fict.itmo.ru) \
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming) \
Year: 2023/2024 \
Group: K34202 \
Author: Shabashov Vasiliy Andreevich \
Lab: Lab2 \
Date of create: 27.11.2023

![1](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/6f44733a-1265-4084-9ebf-fd3a82ae72ad)
![2](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/56acebf7-f21a-4e9f-bccd-27097dc0fac6)
![3](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/9bcfaf35-d532-4ab0-8c5c-d4f93b944c65)
![4](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/83a9caec-b7b9-4037-bdb1-bba13ccee599)
![5](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/e49be6be-6e70-4805-a7d6-ffa35f897d1e)
![6](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/06e1d7c1-7008-4945-b07e-01448b73b259)


# Лабораторная работа №2 "Развертывание дополнительного CHR, первый сценарий Ansible"

## Цель работы
С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.

## Ход работы

Мы произвели установку второго CHR на компьютере и создали второй клиент OVPN на этом CHR, аналогично установке первого CHR. \
Далее установили библиотеку Ansible, чтобы работать по протоколу SSH.

Создали файл hosts.ini, в котором прописали конфигурационные настройки для обоих CHR. В этом файле мы указали адреса и учетные данные для подключения к каждому CHR, чтобы иметь возможность управлять ими при помощи Ansible. \
Проверили подключение командой ```ansible -i hosts.ini -m ping chr```
```
[chr]
chr1 ansible_host=172.27.224.3
chr2 ansible_host=172.27.224.46

[chr:vars]
ansible_connection=ansible.netcommon.network_cli
ansible_network_os=community.routeros.routeros
ansible_user=admin
ansible_ssh_pass=admin
```
![hosts](1.png)

Далее был создан и выполнен файл ansible_playbook.yml с настройками конфигурации.
```
---
- name:  CHR setting
  hosts: chr
  tasks:
    - name: Create users
      routeros_command:
        commands: 
          - /user add name=name group=read password=name

    - name: Create NTP client
      routeros_command:
        commands:
          - /system ntp client set enabled=yes server=0.ru.pool.ntp.org
        
    - name: OSPF with router ID
      routeros_command:
        commands: 
          - /interface bridge add name=loopback
          - /ip address add address=172.16.0.1 interface=loopback network=172.16.0.1
          - /routing id add disabled=no id=172.16.0.1 name=OSPF_ID select-dynamic-id=""
          - /routing ospf instance add name=ospf-1 originate-default=always router-id=OSPF_ID
          - /routing ospf area add instance=ospf-1 name=backbone
          - /routing ospf interface-template add area=backbone auth=md5 auth-key=admin interface=ether1

    - name: Get facts
      routeros_facts:
        gather_subset:
          - interfaces
      register: output_ospf

    - name: Print output
      debug:
        var: "output_ospf"
```
![playbook](2.png)

Проверяем связь NTP Client.

![NTP Client](3.png) \
![NTP Client](5.png) \
![NTP Client](4.png) \
![NTP Client](6.png)

## Вывод
С использованием Ansible мы успешно произвели настройку нескольких сетевых устройств и получили информацию о них. Создали файл Inventory, в котором хранятся данные о конфигурации каждого устройства. В этом файле мы указали адреса, учетные данные и другие параметры, необходимые для работы с каждым устройством при помощи Ansible.

[Конфигурация устройства CHR 1](routerOS1.txt) \
[Конфигурация устройства CHR 2](routerOS2.txt)

![Scheme](scheme.png)


# 2023-11-21 16:27:25 by RouterOS 7.11.2
# software id = 
#
/interface bridge
add name=loopback
/interface ovpn-client
add certificate=name_1 cipher=aes256-cbc connect-to=62.84.126.166 \
    mac-address=02:31:44:0E:5A:D6 name=ovpn-out1 port=443 user=name_2
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing id
add disabled=no id=172.16.0.1 name=OSPF_ID select-dynamic-id=""
/routing ospf instance
add disabled=no name=ospf-1 originate-default=always router-id=OSPF_ID
/routing ospf area
add disabled=no instance=ospf-1 name=backbone
/ip address
add address=172.16.0.1 interface=loopback network=172.16.0.1
/ip dhcp-client
add interface=ether1
/routing ospf interface-template
add area=backbone auth=md5 auth-key=admin disabled=no interfaces=ether1
/system note
set show-at-login=no
/system ntp client
set enabled=yes
/system ntp client servers
add address=0.ru.pool.ntp.org


# 2023-11-21 16:27:49 by RouterOS 7.11.2
# software id = 
#
/interface bridge
add name=loopback
/interface ovpn-client
add certificate=name_1 cipher=aes256-cbc connect-to=62.84.126.166 \
    mac-address=02:F6:5A:16:F4:3E name=ovpn-out1 port=443 user=name
/disk
set slot1 slot=slot1 type=hardware
set slot2 slot=slot2 type=hardware
set slot3 slot=slot3 type=hardware
set slot4 slot=slot4 type=hardware
set slot5 slot=slot5 type=hardware
set slot6 slot=slot6 type=hardware
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/routing id
add disabled=no id=172.16.0.1 name=OSPF_ID select-dynamic-id=""
/routing ospf instance
add disabled=no name=ospf-1 originate-default=always router-id=\
    OSPF_ID
/routing ospf area
add disabled=no instance=ospf-1 name=backbone
/ip address
add address=172.16.0.1 interface=loopback network=172.16.0.1
/ip dhcp-client
add interface=ether1
/routing ospf interface-template
add area=backbone auth=md5 auth-key=admin disabled=no interfaces=\
    ether1
/system note
set show-at-login=no
/system ntp client
set enabled=yes
/system ntp client servers
add address=0.ru.pool.ntp.org


![scheme](https://github.com/vshabashov/2023_2024-network_programming-k34202-Shabashov-Vasiliy-Andreevich/assets/157371606/2a4bfa8f-dbf9-4fa7-a904-6c976925763e)
