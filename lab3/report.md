University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2024/2025  
Group: K34212  
Author: Kardakov Maxim Dmitrievich  
Lab: Lab3  
Date of create: 15.12.2024  
Date of finished: .12.2024 

## Отчёт о Лабораторной работе №3 <br>"Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"

### Описание работы

В данной лабораторной работе вы ознакомитесь с интеграцией Ansible и Netbox и изучите методы сбора информации с помощью данной интеграции.


### Цель работы

С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.


### Ход работы

## 1. Поднять Netbox на дополнительной VM.  

Для поднятия Netbox, на сервере были установлены redis, postgresql и докер. В postgresql была создана БД, а в redis был проверен пинг.  
![image](https://github.com/user-attachments/assets/c954447e-18f7-4856-add4-34ded25f5c8d)  
![image](https://github.com/user-attachments/assets/d20401a9-ae25-4282-94f7-08c20b3a606a)  


## 2. Заполнить всю возможную информацию о ваших CHR в Netbox.  

## 3. Используя Ansible и роли для Netbox в тестовом режиме сохранить все данные из Netbox в отдельный файл, результат приложить в отчёт.  

## 4. Написать сценарий, при котором на основе данных из Netbox можно настроить 2 CHR, изменить имя устройства, добавить IP адрес на устройство.  

## 5. Написать сценарий, позволяющий собрать серийный номер устройства и вносящий серийный номер в Netbox.  
