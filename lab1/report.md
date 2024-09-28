University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2024/2025  
Group: K34212  
Author: Kardakov Maxim Dmitrievich  
Lab: Lab1  
Date of create: 28.09.2024  
Date of finished: 28.09.2024  

## Отчёт о Лабораторной работе №1 <br>"Установка CHR и Ansible, настройка VPN"

### Описание работы

Данная работа предусматривает обучение развертыванию виртуальных машин (VM) и системы контроля конфигураций Ansible а также организации собственных VPN серверов.


### Цель работы

Целью данной работы является развертывание виртуальной машины на базе платформы Microsoft Azure с установленной системой контроля конфигураций Ansible и установка CHR в VirtualBox.


### Ход работы

#### 1. Развёртка и настройка виртуальной машины с помощью Yandex Compute Cloud

В качестве ресурса для развёртки виртуальной машины в данной работы был выбрал сервис Yandex Cloud, а именно функционал Compute Cloud. Были выбраны минимальные настройки виртуальной машины, в качестве ОС - Ubuntu 24.04 LTS. **Интересно отметить, что Yandex Cloud полностью перешёл на подключение через ssh, так как, по их словам, "Подключение по логину и паролю является устаревшим и небезопасным".** На моей рабочей машине были сгенерированы ключи ssh через ```ssh keygen -t```, после чего виртуальная машина была успешно создана и запущена.  
<img src="https://github.com/user-attachments/assets/28293214-fbae-473a-bcbf-31eb75b74bb9" alt="Созданная виртуальная машина" width="600"/>  
Далее было произведено успешное подключение к виртуальной машине через консоль.
<img src="https://github.com/user-attachments/assets/2f6a846f-7f9d-4df8-aec9-05794f925e92" alt="Первое подключение" width="600"/>  
После чего были установлены python3.12 и ansible. Однако, в Ubuntu 24 был вшит новый механизм управления окружениями в Python, который предотвращал установку пакетов с помощью pip напрямую в системное окружение. Можно было сделать виртуальное окружение, но я поступил более решительно - исполнил команду ```sudo apt install ansible```. Результаты установки на скринах ниже.  
<img src="https://github.com/user-attachments/assets/46fbc708-fc33-4fbd-a988-13f2da8052be" alt="Установленный python" width="600"/>  
<img src="https://github.com/user-attachments/assets/209cd0fc-905d-42a2-9d95-b92a0937d9a9" alt="Установленный ansible" width="750"/>  


#### 2. Установка RouterOS на VirtualBox. Настройка WireGuard клиента

Для следующего этапа с официального сайта [mikrotik](https://mikrotik.com/download) был скачен необходимый *.vdi* файл. В VirtualBox были выставлены все необходимые настройки, в качестве носителя был выбран скаченный ранее файл RouterOS.  
<img src="https://github.com/user-attachments/assets/7bb454c0-2d0d-41af-9af5-edfd76262295" alt="Создание виртуальной машины" width="600"/>   
Далее виртуальная машина была успешно запущена, с помощью команды  ```ip address print``` был получен IP-адрес роутера.  
<img src="https://github.com/user-attachments/assets/6bf9f9b9-e213-4293-a20d-e80aa35adca8" alt="Работающая виртуалка" width="600"/>  
После чего, с помощью WinBox, загруженного с того же сайта mikrotik, был открыт интерфейс виртуального роутера. Так как начиная с RouterOS 7.1 в прошивке уже имеется WireGuard, сразу же был создан необходимый интерфейс.
<img src="https://github.com/user-attachments/assets/4fb355ae-c476-48d3-b71b-8227fe39424f" alt="Создание интерфейса для WireGuard" width="900"/>  
Следующим шагом на созданный интерфейс был назначен адрес ```10.1.1.1/24```.
<img src="https://github.com/user-attachments/assets/d5b00bfd-6095-466d-952a-13e7c2c9e514" alt="Назначение IP-адреса для интерфейса" width="500"/>   


#### 3. Настройка WireGuard сервера

Возвращаясь к облачной виртуальной машине, переходим к настройке серверной части нашего VPN-туннеля. Первым делом был установлен wireguard.  
<img src="https://github.com/user-attachments/assets/09a4b804-b7e9-48a3-be0a-5fc012b48992" alt="Установленный WireGuard" width="600"/>  
Далее были сгенерированы публичный и приватный ключи.  
<img src="https://github.com/user-attachments/assets/65c1a8f8-d542-4280-b782-bf6dac62046f" alt="Создание ключей" width="750"/>  
После этого с помощью команды ```sudo nano /etc/wireguard/wg0.conf``` был создан конфигурационный файл со следующими настройками:  
```
[Interface]

PrivateKey = uIGImemVXD7lxAJ6e4WjwWwMpoaJINUqIavntSjrjk8=
Address = 10.1.1.2/24
ListenPort = 13231

[Peer]

PublicKey = RWHSuA8UumBZwh2Wp7bg2zWQL1QXtMgt2QDX7ox2qEM=
AllowedIPs = 10.1.1.1/24
```
В данном файле в строку приватного ключа был записан приватный ключ самого сервера, а в качестве публичного ключа пира - публичный ключ клиента, то есть роутера. Порт для прослушивания и пул адресов так же были взяты из предыдущей настройки роутера.  
После этого был настроен пир для интерфейса WireGuard на роутере. В качестве эндпоинта был установлен публичный IP4-адрес сервера (облачной виртуальной машины), в качестве публичного ключа - публичный ключ того же сервера.   
<img src="https://github.com/user-attachments/assets/91a13a03-0b20-4184-93cd-497bd4dc9560" alt="Настройка пира на роутере" width="600"/>  
После этого на сервере был запущен wireguard с помощью команды ```sudo systemctl start wg-quick@wg0.service```, статус был проверен с помощью команды ```sudo systemctl status wg-quick@wg0.service```.  
<img src="https://github.com/user-attachments/assets/938f0024-0051-42b0-ba48-4bd4a2d11e02" alt="Успешный запуск wireguard-сервера" width="750"/> 


#### 4. Проверка доступа 

Был проведён пинг с сервера на клиента, с клиента на сервер, а также пинг 8.8.8.8 (dns.google) с каждого из устройств. Результаты приведены на скриншотах.  
<img src="https://github.com/user-attachments/assets/a5270135-62ff-4048-aef0-4763e7257bd0" alt="Пинг сервер-клиент" width="600"/>   
<img src="https://github.com/user-attachments/assets/1cd3dd32-e8e2-4d64-a818-d0a635d10310" alt="Пинг сервер-гугл" width="600"/>   
<img src="https://github.com/user-attachments/assets/31afa57c-1c63-4775-8b82-c0c9036524b3" alt="Пинг клиент-сервер" width="600"/>   
<img src="https://github.com/user-attachments/assets/f78d79ef-e59f-49ed-97b7-8808c4e686ff" alt="Пинг клиент-гугл" width="600"/>  


### Вывод

В данной лабораторной работе была развернута облачная виртуальная машина с OS Ubuntu 24.04 (а также с ansible и python), на которой был настроен WireGuard-сервер, после чего была создана виртуальная машина RouterOS, на которой был настроен WireGuard-клиент. В результате был установлен VPN-туннель между данными устройствами. Была проведена проверка доступности, результаты которой свидетельствуют о правильности выполнения работы. 
