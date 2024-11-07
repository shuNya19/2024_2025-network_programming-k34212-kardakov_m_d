University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2024/2025  
Group: K34212  
Author: Kardakov Maxim Dmitrievich  
Lab: Lab2  
Date of create: 24.10.2024  
Date of finished: 07.11.2024  

## Отчёт о Лабораторной работе №2 <br>"Развертывание дополнительного CHR, первый сценарий Ansible"

### Описание работы

В данной лабораторной работе вы на практике ознакомитесь с системой управления конфигурацией Ansible, использующаяся для автоматизации настройки и развертывания программного обеспечения.


### Цель работы

С помощью Ansible настроить несколько сетевых устройств и собрать информацию о них. Правильно собрать файл Inventory.


### Ход работы

#### 1. Установить второй CHR на своем ПК.

На пк была создана ещё одна виртуальная машина с RouterOS.  

<img src="https://github.com/user-attachments/assets/ab73655a-b6e6-4bc6-ab9a-46e25a05a9b3" alt="Созданная виртуальная машина" width="600"/>  

#### 2. Организовать второй OVPN Client на втором CHR.

Далее необходимо было настроить второй WireGuard Client на созданной ВМ. Как и в предыдущей работе, был создан интерфейс, пир, а также были настроены IP-адреса для вайргарда.  
Ниже представлены скриншоты конфигурации с обоих роутеров.  
<img src="https://github.com/user-attachments/assets/49b48c7d-7567-4cf8-8f82-97444ff0a8a6" alt="Адрес-листы на обоих роутерах" width="600"/>  
<img src="https://github.com/user-attachments/assets/172eaeba-42eb-4ebd-8b73-3ed813977bb4" alt="Пиры на обоих роутерах" width="800"/>  
Был изменен конфигурационный файл VPN-сети на VPN-сервере (виртуалке в облаке), а именно был добавлен новый пир.

```
[Interface]

PrivateKey = <тут приватный ключ>
Address = 10.1.1.1/24
ListenPort = 13231

[Peer]

PublicKey = RWHSuA8UumBZwh2Wp7bg2zWQL1QXtMgt2QDX7ox2qEM=
AllowedIPs = 10.1.1.2/32

[Peer]

PublicKey = irvg3Cmv5T1NOf0FYdek2RDFQ5/1BfeKktAvZJj80Fs=
AllowedIPs = 10.1.1.3/32
```

После этого VPN-сервер был перезапущен. Было проверено подключение.  
<img src="https://github.com/user-attachments/assets/9134719a-2eb4-48b4-af2f-bb37a3fe599f" alt="С роутеров на сервер" width="900"/>  
<img src="https://github.com/user-attachments/assets/89c9f89f-4bb2-48e4-9338-973a4c185df1" alt="С сервера на роутеры" width="600"/>   

#### 3. Используя Ansible, настроить NTP, OSPF и данные нового пользователя сразу на 2-х CHR.

Ansible был установлен на машину в предыдущей работе. Первым пунктом был создан файл inventory, содержащий конфигурационную информацию для роутеров 1 и 2.  
<img src="https://github.com/user-attachments/assets/a0bfd0ef-bbe6-4ce1-aa6b-0472e0bfe109" alt="Файл inventory" width="900"/>  
Файл был проверен с помощью команды ```ansible-inventory --list -l```  
<img src="https://github.com/user-attachments/assets/1ddb7cb9-4c3e-4ff1-a27f-17c62015f5e0" alt="Проверка содержимого файла inventory" width="600"/> 

Далее необходимо было настроить ssh соедниение между сервером и роутерами: это необходимо для работы Ansible. Для этого на роутерах был включен ssh, на сервере были сгенерированы ключи, которые были перекинуты на роутеры.  
<img src="https://github.com/user-attachments/assets/ba1e5420-674f-47c2-a581-677038e9474d" alt="Включение поддержки SSH" width="400"/>  
<img src="https://github.com/user-attachments/assets/5406dceb-fcab-4c2a-83d4-c8de664381bd" alt="Генерация ключей на сервере" width="600"/>  
<img src="https://github.com/user-attachments/assets/382dcb3f-9d7d-407f-894f-0bb0d568d7b1" alt="Импорт ключей на роутер" width="600"/>  
<img src="https://github.com/user-attachments/assets/f70b95c3-1383-4d67-8117-702c41cf77a3" alt="Установка ключей на роутер" width="800"/>  

После этого была проверена доступность пути от сервера до роутеров через Ansible.  
<img src="https://github.com/user-attachments/assets/77c02ada-cb56-44dd-bd8d-29a6c463ce96" alt="Проверка доступности" width="600"/>  

После этого был написан код скрипта .yml, который и будет выполнять настройку роутеров.  
```
- name: "Configure MikroTik routers"
  hosts: mikrotik
  gather_facts: no

  tasks:
    - name: Set User&Password
      community.routeros.command:
        commands: "user add name=ansible_user password=ansible_user group=full"

    - name: Configure NTP
      community.routeros.command:
        commands: "system ntp client set enabled=yes servers=8.8.8.8"

    - name: Configure OSPF
      community.routeros.command:
        commands:
          - /interface bridge add name=Lo
          - /ip address add address="{{ router_id }}"/32 interface=Lo
          - /routing ospf instance add name=v2inst version=2 router-id="{{ router_id }}"
          - /routing ospf area add name=backbone_v2 area-id=0.0.0.0 instance=v2inst
          - /routing ospf interface-template add network=0.0.0.0/0 area=backbone_v2

    - name: Gather OSPF information
      community.routeros.command:
        commands: "/routing ospf neighbor print"
      register: ospf_info

    - name: Gather config information
      community.routeros.facts:
        gather_subset:
          - config
      register: config_info

    - name: Get OSPF information
      debug:
        msg: "{{ ospf_info }}"

    - name: Get config information
      debug:
        msg: "{{ config_info }}"
```
Также был изменен файл inventory - был добавлен router-id.  
```
[mikrotik]
ansible_host=10.1.1.2 router_id = 1.1.1.1 ansible_user=admin ansible_password=admin ansible_connection=network_cli ansible_network_os=routeros
ansible_host=10.1.1.3 router_id = 2.2.2.2 ansible_user=admin ansible_password=admin ansible_connection=network_cli ansible_network_os=routeros
```

Первый запуск плейбука завершился ошибкой, так как не был установлен модуль paramiko.  
<img src="https://github.com/user-attachments/assets/255c6b20-9738-4415-a238-8c742e4bd35e" alt="Ошибка выполнения" width="600"/>  

Но уже следующий запуск (после установки paramiko) прошёл успешно.  
<img src="https://github.com/user-attachments/assets/c20e8290-48b1-41de-b1d7-ddde6325804a" alt="Успешное выполнение плейбука" width="600"/>  
<img src="https://github.com/user-attachments/assets/d041e105-b528-4880-88f6-3854016f9122" alt="Общие результаты выполнения плейбука" width="800"/>  

После этого была проведена проверка работы NTP и OSPF (настроенных с помощью Ansible). Всё прошло успешно.  
<img src="https://github.com/user-attachments/assets/5e5a0eeb-58a8-40da-8785-d10a516c6a15" alt="Проверка NTP" width="600"/>   
<img src="https://github.com/user-attachments/assets/73856795-6bd4-4093-99e2-b3475ec6052c" alt="Проверка OSPF" width="600"/>  

### Вывод

В данной работе была проведена настройка двух роутеров MikroTik с помощью специально написанного плейбука Ansible. То есть фактически запуском одного скрипта была проведена поставка ПО на несколько машин, которая увенчалась успехом.

