# TryHackMe: Wgel CTF

## Task 1: Wgel CTF

Используем nmap для сканирования машины:
```sh
nmap -sC -sV 10.10.145.87
```
![ScreenShot](screenshots/1.png)

Мы нашли:
- 22 port - SSH (OpenSSH 7.2p2) 
- 80 port - HTTP (Apache httpd 2.4.18)

Перейдем на сайт:

![ScreenShot](screenshots/2.png)

Попробуем найти другие директории сайта:
```sh
gobuster dir -u http://10.10.145.87/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![ScreenShot](screenshots/3.png)

Находим директорию **/sitemap**:

![ScreenShot](screenshots/4.png)

На сайте ничего интересного найдено не было, поэтому попробуем найти другие директории относительно найденной до этого:
```sh
gobuster dir -u http://10.10.145.87/sitemap -w /usr/share/wordlists/dirb/common.txt -x php,html,txt
```

![ScreenShot](screenshots/5.png)

Перейдем в директорию **/.ssh**:

![ScreenShot](screenshots/6.png)

Находим приватный RSA-ключ:

![ScreenShot](screenshots/7.png)

Копируем его в файл:

![ScreenShot](screenshots/8.png)

Мы нашли ключ, а логин пользователя нам неизвестен. Ищем его среди директорий. Находим логин **jessie** в Apache Default Page.

![ScreenShot](screenshots/9.png)

Присваиваем нужные права файлу и подключаемся по SSH:

![ScreenShot](screenshots/10.png)

Находим первый флаг:

![ScreenShot](screenshots/11.png)

### Question 1: User flag - 057c67131c3d5e42dd5cd3075b198ff6

Посмотрим, с чем мы можем взаимодействовать, используя sudo:

![ScreenShot](screenshots/12.png)

Находим команду **wget**. Посмотрим на GTFORBins:

![ScreenShot](screenshots/13.png)

![ScreenShot](screenshots/14.png)

Чтобы получить файл, включим прослушивание порта, причем именно 80 - передача проходит по протоколу HTTP:

![ScreenShot](screenshots/15.png)

Передаем файл **/etc/passwd**:

![ScreenShot](screenshots/16.png)

![ScreenShot](screenshots/17.png)

Сохраним содержимое в отдельный файл с названием **passwd** (чтобы потом заменить этот файл на взламываемой машине):

![ScreenShot](screenshots/18.png)

Создадим пароль, причем зашифрованный:

![ScreenShot](screenshots/19.png)

Вставим созданный пароль в созданный файл:

![ScreenShot](screenshots/20.png)

Откроем порт 8080 посредством http.server:

![ScreenShot](screenshots/21.png)

Заменим файл на удаленной машине:

![ScreenShot](screenshots/22.png)

![ScreenShot](screenshots/23.png)

Теперь мы можем перейти в root-пользователя, используя пароль, хэш которого мы внесли в файл:

![ScreenShot](screenshots/24.png)

Находим root-флаг:

![ScreenShot](screenshots/25.png)

### Question 2: Root flag - b1b968b37519ad1daa6408188649263d
