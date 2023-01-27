# TryHackMe: Bounty Hacker

## Task 1: Living up to the title
### Question 1: Find open ports on the machine
Используем nmap для сканирования машины:
```sh
nmap -sC -sV 10.10.241.85
```
![ScreenShot](screenshots/1.png)

Найденные порты:
- 21 port - FTP (vsftpd 3.0.3)
- 22 port - SSH (OpenSSH 7.2p2)
- 80 port - HTTP (Apache httpd 2.4.18)

### Question 2: Who wrote the task list?
Попробуем зайти на сайт, который находится на 80 порте машины.

![ScreenShot](screenshots/2.png)

К сожалению, ничего существенного не находим, поэтому пробуем найти скрытые директории, при помощи инструмента *dirbuster*

![ScreenShot](screenshots/3.png)

![ScreenShot](screenshots/4.png)

В итоге находим директорию */images/*, если перейти по этому пути, можно обнаружить список файлов, находящихся на веб-сервере.

![ScreenShot](screenshots/5.png)

Далее, попробуем зайти на FTP-сервер под анонимом и у нас это получается:

![ScreenShot](screenshots/6.png)

В таком случае, посмотрим на список файлов:

![ScreenShot](screenshots/7.png)

Мы нашли 2 txt-файла. Сразу скачиваем их и смотрим:

![ScreenShot](screenshots/8.png)

![ScreenShot](screenshots/9.png)
- task.txt - план каких-то действий, а также имя автора - lin
- locks.txt - по всей видимости пароли

### Question 3: What service can you bruteforce with the text file found?

Имея пароль и логин, мы могли бы попытаться войти по ssh, но правильного пароля у нас нет, поэтому при помощи инструмента *hydra*, сделаем перебор паролей:
```sh
hydra -l lin -P locks.txt 10.10.241.85 ssh
```
![ScreenShot](screenshots/10.png)

### Question 4: What is the users password?
При помощи инструмент *hydra* нам удается найти пароль - ***RedDr4gonSynd1cat3***

### Question 5: user.txt
Теперь, имея логин и пароль, попробуем подключиться по ssh:

![ScreenShot](screenshots/11.png)

Просматриваем файлы в текущей директории, тем самым находим один из нужных нам файлов *user.txt*. Читаем его:

![ScreenShot](screenshots/12.png)

### Question 6: root.txt
Следующим этапом запустим соответствующую команду, чтобы узнать, что рядовой пользователь может делать наряду с суперпользователем:
```sh
sudo -l
```
![ScreenShot](screenshots/13.png)

Видим, что нам доступен tar (утилита tar предназначена для создания архивов файлов и каталогов). Находим на GTFORBins раздел с tar. Нам нужна графа *Sudo*:

![ScreenShot](screenshots/14.png)

Применяем найденную команду, а также проверяем, кем мы являемся:
```sh
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
![ScreenShot](screenshots/15.png)

Перемещаемся в директорию */root*, где находим второй нужный нам файл, а в нем и заключительный флаг суперпользователя:

![ScreenShot](screenshots/16.png)
