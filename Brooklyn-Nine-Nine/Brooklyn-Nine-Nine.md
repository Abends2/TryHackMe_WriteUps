# TryHackMe: Brooklyn Nine Nine

## Task 1: Deploy and get hacking 
Используем nmap для сканирования машины:
```sh
nmap -sC -sV 10.10.81.234
```
![ScreenShot](screenshots/1.png)

Мы нашли:
- 21 port - FTP (vsftpd 3.0.3)
- 22 port - SSH (OpenSSH 7.6p1)
- 80 port - HTTP (Apache httpd 2.4.29)

Осмотрим сайт:

![ScreenShot](screenshots/2.png)

К сожалению, нас сайте нет ничего, кроме картинки, поэтому проверим файл на FTP-сервере:

![ScreenShot](screenshots/3.png)

![ScreenShot](screenshots/4.png)

В сообщении сказано, что у **jake** слабый пароль, который ему советуют поменять. Попробуем сбрутить пароль при помощи инструмента hydra. Брутить будем относительно SSH:

![ScreenShot](screenshots/5.png)

В итоге, получаем следующие данные для подключения по SSH - **jake:987654321**:

![ScreenShot](screenshots/6.png)

Ищем user-флаг:

![ScreenShot](screenshots/7.png)

### Question 1: User flag - ee11cbb19052e40b07aac0ca060c23ee

Наши права на sudo:

![ScreenShot](screenshots/8.png)

Нам доступная утилита **less**. Посмотрим, можно ли с ее помощью получить root'а:

![ScreenShot](screenshots/9.png)

Следуем способу выше:

![ScreenShot](screenshots/10.png)

Как итог, повышаем привилегии и находим финальный флаг:

![ScreenShot](screenshots/11.png)

### Question 2: Root flag - 63a9f0ea7bb98050796b649e85481845
