University: [ITMO University](https://itmo.ru/ru/)  
Faculty: [FICT](https://fict.itmo.ru)  
Course: [Network programming](https://github.com/itmo-ict-faculty/network-programming)  
Year: 2024/2025  
Group: K34212  
Author: Kardakov Maxim Dmitrievich  
Lab: Lab3  
Date of create: 15.12.2024  
Date of finished: 24.02.2025 

## Отчёт о Лабораторной работе №3 <br>"Развертывание Netbox, сеть связи как источник правды в системе технического учета Netbox"

### Описание работы

В данной лабораторной работе вы ознакомитесь с интеграцией Ansible и Netbox и изучите методы сбора информации с помощью данной интеграции.


### Цель работы

С помощью Ansible и Netbox собрать всю возможную информацию об устройствах и сохранить их в отдельном файле.


### Ход работы

#### 1. Поднять Netbox на дополнительной VM.  

Для поднятия Netbox, на сервере были установлены redis, postgresql и докер. В postgresql была создана БД, а в redis был проверен пинг.<br/><br/>
![image](https://github.com/user-attachments/assets/c954447e-18f7-4856-add4-34ded25f5c8d)<br/><br/>
![image](https://github.com/user-attachments/assets/d20401a9-ae25-4282-94f7-08c20b3a606a)<br/><br/>

Далее был склонирован репозиторий netbox. Был создан новый пользователь, а также был сгенерирован секретный ключ<br/><br/>

![image](https://github.com/user-attachments/assets/046fde5d-b02a-4240-bc5f-3c2a38cd1bab)<br/><br/>
![image](https://github.com/user-attachments/assets/264e3c18-a851-4aa1-9c52-9c2c41bf413f)<br/><br/>
![image](https://github.com/user-attachments/assets/db9f3436-0220-4c59-a533-79c1c6a0e626)<br/><br/>

В базу данных Postgress была добавлена актуальная информация, включая секретный ключ<br/><br/>

![image](https://github.com/user-attachments/assets/e2da21d1-707e-497c-815f-db8c2ba3550b)<br/><br/>
![image](https://github.com/user-attachments/assets/2593bedc-d998-486b-8804-faa02386755a)<br/><br/>

После этого был создан файл docker-compose.yml. Были подняты три контейнера: Netbox, Redis и Postgres. *Тут я перешёл на другой хостинг, поэтому консоль так поменялась. Яндекс сервер 2к в месяц...)* <br/><br/>

![image](https://github.com/user-attachments/assets/e166a825-7f66-493a-ba35-01a500739a67)<br/><br/>
![image](https://github.com/user-attachments/assets/5ed81470-8bf1-45d2-82a0-a35bf4467502)<br/><br/>

Доступ к Netbox был успешно получен<br/><br/>

![image](https://github.com/user-attachments/assets/a76fa729-c18e-43bb-8f5e-00283924cc9f)<br/>

#### 2. Заполнить всю возможную информацию о ваших CHR в Netbox.  

Для того, чтобы добавить устройство Netbox, необходимо:
- Создать производителя
- Создать модель
- Создать сайт

После прохождения этих этапов, успешно были созданы CHR1 и CHR2<br/><br/>
![image](https://github.com/user-attachments/assets/30cd5cb9-a0ed-4db4-a5b9-86e9ea8bf4e2)<br/><br/>
Далее для каждого из устройств были добавлены интерфейсы и выданы IP-адреса<br/><br/>
![image](https://github.com/user-attachments/assets/3a134bc8-c59b-4164-9672-c2a9e243b7bb)<br/><br/>
![image](https://github.com/user-attachments/assets/7ea28ab7-6b79-4f5a-9901-d0f1d64e5be8)<br/>

#### 3. Используя Ansible и роли для Netbox в тестовом режиме сохранить все данные из Netbox в отдельный файл, результат приложить в отчёт.  

В первую очередь, для того, чтобы наша виртуалка смогла изменить что-то в Netbox, нужно было получить API-токен в профиле<br/><br/>
![image](https://github.com/user-attachments/assets/c15a1dab-5505-43a6-bf0e-433a1687f36b)<br/><br/>
Далее был написан yml файл, который и будет вытаскивать данные<br/><br/>
![image](https://github.com/user-attachments/assets/39fd0672-604f-4279-b956-6bc5bcc60e58)<br/><br/>
С данным этапом не было никаких проблем: команды была испешно выполнена, результаты работы были записаны в файл netbox_inventory.yml<br/><br/>
![image](https://github.com/user-attachments/assets/ea9876d3-7801-4b3d-a1bb-8f948e24b4ab)<br/><br/>
Полный файл netbox_inventory.yml прикрепляю в папку lab3.

#### 4. Написать сценарий, при котором на основе данных из Netbox можно настроить 2 CHR, изменить имя устройства, добавить IP адрес на устройство.  

Во-первых, для данной задачи был немного доработан файл netbox_inventory.yml, так как необходимо добавить инструкции для работы Ansible, а также указать айпишники внутри VPN-сети (ansible_host)<br/><br/>
![image](https://github.com/user-attachments/assets/cadd2c43-1a22-4eff-8491-dc62bfa81fd4)<br/><br/>
После этого был создан файл, который и должен был изменить IP и имена роутеров<br/><br/>
![image](https://github.com/user-attachments/assets/9f0923a6-0d69-4431-853e-9967a6f58d13)<br/><br/>
Однако первый запуск закончился ошибкой. <br/><br/>
![image](https://github.com/user-attachments/assets/c9e0d26b-8e41-4c1c-8d84-56d098cc56ba)<br/><br/>
Проблема возникала из-за того, что Ansible не находит подходящий модуль для сбора фактов для RouterOS. В случае с RouterOS стандартный модуль фактов не подходит, поэтому нужно было отключить сбор фактов, прописав `gather_facts: false` в плейбуке. Но это не было единственной ошибкой: после добавления данной строки, появилась ошибка аутентификации. В интернете я нашёл, что это происходит из-за разного подхода paramiko и pylibssh, и предпочитетльнее использовать pylibssh. Подгрузив его в виртуальную среду, задача наконец была выполнена успешно.<br/><br/>
![image](https://github.com/user-attachments/assets/8aaaa91a-d03c-40c4-a939-85adbfa71bdc)<br/><br/>
![image](https://github.com/user-attachments/assets/dbff632f-28f2-480b-abcf-ca8b2ee274e4)<br/><br/>
![image](https://github.com/user-attachments/assets/c7c399dd-7722-49bb-9bd9-8f0203c50eee)<br/>

#### 5. Написать сценарий, позволяющий собрать серийный номер устройства и вносящий серийный номер в Netbox.  

Был написан похожий файл serial_number.yml, который тянет имя устройства и его серийный номер из роутеров, после чего стучиться в netbox и изменяет значения серийных номеров в соответствии с именами (благодаря прошлому заданию мы точно знаем, что имена в netbox и на самих микротиках идентичны)<br/><br/>

![image](https://github.com/user-attachments/assets/19fb796e-9e03-44cb-b57a-4e9b4ee41a40)<br/><br/>
![image](https://github.com/user-attachments/assets/8f5c9aab-0dfa-4ae3-8704-87e4160d329a)<br/><br/>
![image](https://github.com/user-attachments/assets/a81ededb-cf57-45fe-a589-940a6dc7a0cc)<br/>

### Вывод

В данной работе было проведено знакомство с Netbox, после чего была реализована автоматизированная система учета сетевых устройств с возможностью двусторонней синхронизацией между Netbox и реальной инфраструктурой.
