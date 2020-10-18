# Администрирование Linux. ДЗ №3
## Работа с пользователями


## Цель:
> Работа состоит из 2х пунктов. `* не обязательна к выполнению.`
> 1. Первая часть:
>  * Создать нескольких пользователей, задать им пароли, домашние директории и шеллы;
>  * Создать группу admin;
>  * Включить нескольких из ранее созданных пользователей, а также пользователя root, в группу admin;
>  * Запретить всем пользователям, кроме группы admin, логин в систему по SSH в выходные дни (суббота и воскресенье, без учета праздников).
>  * \* С учётом праздничных дней.
> (Для упрощения проверки можно разрешить парольную аутентификацию по SSH и использовать ssh user@localhost проверяя логин с этой же машины)
> ---
> 2. Вторая часть:
>  * Установить docker;
>  * Дать конкретному пользователю:
>    * права работать с docker (выполнять команды docker ps и т.п.);
>    * \* возможность перезапускать демон docker (systemctl restart docker) не выдавая прав более, чем для этого нужно;
---


## Часть 1

<details>
<summary>Сама работа</summary>

---
### 1. Создание нескольких пользователей

Чтобы создать пользователя в Linux используется команда `useradd <опции> <имя пользователя>`, нам необходимо использовать ключи (опции) `-d /home/<имя пользователя>` для указания домашнего каталога пользователя, и ключ `-s <путь до исполнительного файла оболочки>` для указания используемой оболочки для данного пользователя.


Создадим несколько (3) пользователей с указанием домашней директории и путей для bash (в нашем случае). 
Для этого выполним команды 
```
sudo useradd -d /home/test1 -s /bin/bash test1
sudo useradd -d /home/test2 -s /bin/bash test2
sudo useradd -d /home/test3 -s /bin/bash test3
```

Результат выполнения команд (создание пользователей):

![Результат выполнения команд (создание пользователей)](https://sun9-63.userapi.com/FVJdk6u9kEf1M6tzw7bTk4jIjy2AaZjThsCuzw/wYPOeRXBLWc.jpg "Результат выполнения команд (создание пользователей)")


Для добавления паролей выполним команду `sudo passwd <имя пользователя>` для каждого из пользователей.

Всем пользователям был выставлен незамысловатый пароль: "Qwerty".

Результат выполнения команд (смены пароля для пользователей):

![Результат выполнения команд (смены пароля для пользователей)](https://sun9-8.userapi.com/eP-TZAVCCb4HdXljgfinmMSYGUZYcSVlDGcLnQ/BejFRx0ikRM.jpg "Результат выполнения команд (смены пароля для пользователей)")


Далее была добавлена группа `admin` для этого была выполнена команда `sudo groupadd admin`.
Результат добавления группы:

![Результат добавления группы](https://sun9-59.userapi.com/r6Ah5nZYm5ateNeNOg0oo8o6PoIbmoOUA7Rofg/HL15f3SYDrE.jpg "Результат добавления группы")


Далее были добавлены два пользователя *test1* и *test3* в группу `admin` следующими командами:
```
sudo usermod -aG admin test1
sudo usermod -aG admin test3
```


Также, сразу после добавления удостоверимся в этом, введением команды `id <имя пользователя>`.

Результат добавления в группу нескольких пользователей:

![Результат добавления в группу нескольких пользователей](https://sun9-69.userapi.com/QdWYNILNj1I-CLfdCk4UAtaS-BteYJZABVnhqg/UZHu4u1kNL8.jpg "Результат добавления в группу нескольких пользователей")


Необходимо было еще пользователя **root** добавить в эту же группу:

![Результат добавления пользователя root в группу](https://sun9-38.userapi.com/VYmhf3EatCf7224xnl82PZb6yDW8fZ9K_oIPeg/oZ7tLgPAn2Q.jpg "Результат добавления пользователя root в группу")


При попытках добавить ограничение на использование SSH наткнулся на проблему, что в виртуальной машине, которую я использовал для выполнения лаборатоной работы, не установлен весь пакет SSH (а именно демон не работал и его конфигурационных файлов не было), поэтому была выполнена команда установки всего пакета SSH `sudo apt-get install ssh`

После чего мне удалось выполнить пробный вход в систему под одним из пользователей.

По заданию необходимо было ограничить использование ssh в определенное время:
| **Дн** | **All**  | **test1** | **test2** | **test3** | **root**  | **nikel** |
| -- |:----:| :----:| :----:| :----:| :----:| :----:|
| **Пн** | - | + | - | + | + | - |
| **Вт** | - | + | - | + | + | - |
| **Ср** | - | + | - | + | + | - |
| **Чт** | - | + | - | + | + | - |
| **Пт** | - | + | - | + | + | - |
| **Сб** | + | + | + | + | + | + |
| **Вс** | + | + | + | + | + | + |

Получается, что в файл `sudo vim /etc/security/time/conf` необходимо дописать 
```
sshd;*;test1|test3|root;Al0000-2400
sshd;*;nikel|test2;!Wk0000-2400
```

> Исключение почему-то на моей версии Ubuntu не хотел корректно выполняться (потрачено на разнообразные тесты комбинаций почти 5 часов)

Также, теперь надо добавить в ***/etc/pam.d/sshd*** после последней строчки, которая **начинается** с `auth` следующий код: 
```
account    required     pam_time.so
```


### Проверочка всего, что было выше сотворено:

Удостоверимся, что всё работает корректно.


Попытка войти в понедельник (рабочий день) под разными пользователями:
![Результат впопыток входа](https://sun9-4.userapi.com/FmAtafLjzQqftaqZjYkR0OiG5sE2sAyUYwbTcg/Id_OE7YDOvQ.jpg "Попытка войти в понедельник (рабочий день) под разными пользователями")

Попытка войти в субботу (выходной день) под разными пользователями:
![Результат впопыток входа](https://sun9-31.userapi.com/djo2ZXCkkkVu231gdUPR5CNnq96qoNun5XBx8w/bZLb_eG1zpc.jpg "Попытка войти в субботу (выходной день) под разными пользователями")

Фууух, на этом многострадальная первая часть завершена... Дальше проще

---

</details>


## Часть 2

<details>
<summary>Сама работа</summary>

---
### 1. Установка docker'а


Установка docker'а производилась [по инструкции](https://losst.ru/ustanovka-docker-na-ubuntu-16-04)


Собственно, подтверждение установки docker'а можно считать рисунок, приведенный ниже:



### 2. Выдача прав на работу с docker'ом конкретному пользователю


Выдача прав пользователю *** __ *** производилась командой `sudo `


Попробуем выполнить частоиспользуемые команды работы с docker'ом, такие как:
- `docker ps -a`
- `docker images`
- `docker search`

Собственно, подтверждением _____ можно считать рисунок, приведенный ниже:


---

</details>

## Используемый инструментарий:
- Виртуальная машина под управлением **Ubuntu 18.04.5 LTS**
- GIT (устанавливается командой `sudo apt install git -y`)
- Docker (устанавливается командой `sudo apt install -y docker-ce`) [дополнительная инструкция](https://losst.ru/ustanovka-docker-na-ubuntu-16-04)
- BASH (вроде идет в составе linux, но на всякий случай команда для установки `sudo apt install bash`)
