# TryHackMe: Cyborg

## Task 1: Deploy the machine
### Question 1: Deploy the machine - :heavy\_check_mark:


## Task 2: Compromise the System
Используем nmap для сканирования машины:
```sh
nmap -sC -sV 10.10.82.94
```
![ScreenShot](screenshots/1.png)

Мы нашли:
- 22 port - SSH (OpenSSH 7.2p2)
- 80 port - HTTP (Apache httpd 2.4.18)

### Question 2: Scan the machine, how many ports are open? - 2
### Question 3: What service is running on port 22? - ssh
### Question 4: What service is running on port 80? - http

На порте 80 в корневой директории находится Apache 2 Ubuntu Default Page:

![ScreenShot](screenshots/2.png)

Запустим перебор директорий:
```sh
gobuster dir -u http://10.10.82.94/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![ScreenShot](screenshots/3.png)

Найденные директории:
- /admin
- /etc

Посмотрим найденные директории:

![ScreenShot](screenshots/4.png)

![ScreenShot](screenshots/5.png)

Осмотрим директорию /etc более детально:

![ScreenShot](screenshots/6.png)

![ScreenShot](screenshots/7.png)

![ScreenShot](screenshots/8.png)

Перенесем найденный хэш пароля в текстовый файл для дальнейшего брута:

![ScreenShot](screenshots/9.png)

Вернемся на /admin, на вкладку "Admins":

![ScreenShot](screenshots/10.png)

В админском чате сказано, что существует "music_archive". Попробуем перейти во вкладку "Archive":

![ScreenShot](screenshots/11.png)

Загрузим архив:

![ScreenShot](screenshots/12.png)

Осмотрим содержимое:

![ScreenShot](screenshots/13.png)

![ScreenShot](screenshots/14.png)

![ScreenShot](screenshots/15.png)

Удалось узнать, что backup создан при помощи **Borg Backup**. Мы нашли также в конфиг-файле id и key, что возможно нам понадобится в дальнейшем.

Для того, чтобы открыть (а перед этим монтировать) backup, необходим пароль. Вспоминаем, что мы нашли хэш пароля. Брутим его:

![ScreenShot](screenshots/16.png)

Найденный пароль - **squidward**

Разархивируем скачанный ранее архив:

![ScreenShot](screenshots/17.png)

Проверим, определяется ли backup через borg list:

![ScreenShot](screenshots/18.png)

Монтируем в созданную заранее папку:

![ScreenShot](screenshots/19.png)

![ScreenShot](screenshots/20.png)

Осмотрим файлы, которые находятся в backup'е:

![ScreenShot](screenshots/21.png)

![ScreenShot](screenshots/22.png)

![ScreenShot](screenshots/23.png)

Находим следующие данные - **alex:S3cretP@s3**

Попробуем подключиться по SSH:

![ScreenShot](screenshots/24.png)

После успешного подключения, находим первый флаг:

![ScreenShot](screenshots/25.png)

### Question 5: What is the user.txt flag? - flag{1_hop3_y0u_ke3p_th3_arch1v3s_saf3}

Проверим права, который нам доступны относительно sudo:

![ScreenShot](screenshots/26.png)

Находим bash-скрипт. Его часть на рисунке ниже:

![ScreenShot](screenshots/27.png)

Назначим ему права:

![ScreenShot](screenshots/28.png)

Кстати, скрипт позволяет ввести свою команду при активации, что показано на рисунках ниже:

![ScreenShot](screenshots/29.png)

![ScreenShot](screenshots/30.png)

![ScreenShot](screenshots/31.png)

Как видим, скрипт ответил нам, что мы действуем внутри него от лица root-пользователя. Тем самым, мы можем сразу прочитать root-флаг:

![ScreenShot](screenshots/32.png)

Но такой вариант меня не очень устроил, поэтому вот еще некоторые способы:

![ScreenShot](screenshots/33.png)

![ScreenShot](screenshots/34.png)

В этом способе получается так, что вывод всех введенных команд происходит при завершении сеанса использования олболочки, что тоже не очень удобно, поэтому третий способ:

![ScreenShot](screenshots/35.png)

![ScreenShot](screenshots/36.png)

![ScreenShot](screenshots/37.png)

Тут мы просто меняем .sh скрипт, чтобы выполнилась команда, вызывающая bash-оболочку (от имени root-пользователя).

### Question 6: What is the root.txt flag? - flag{Than5s_f0r_play1ng_H0p£_y0u_enJ053d}
