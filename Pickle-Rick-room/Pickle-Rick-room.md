# TryHackMe: Pickle Rick

## Task 1: Pickle Rick
Используем nmap для сканирования машины:
```sh
nmap -sC -sV 10.10.12.154
```
![ScreenShot](screenshots/1.png)

Мы нашли:
- 22 port - SSH (OpenSSH 7.2p2)
- 80 port - HTTP (Apache httpd 2.4.18)

Вариантов у нас в таком случае немного, поэтому переходим на сайт, который располагается на 80 порте:

![ScreenShot](screenshots/2.png)

Присутствует наличие подсказки на использование Burp Suite. Посмотрим, что и как. Перенаправим запрос в Repeater для удобства и посмотрим, что нам вернет исходный запрос:

![ScreenShot](screenshots/3.png)

Находим ник: **R1ckRul3s**, его можно было найти и в инспекторе браузера. Также находим путь, откуда подгружаются файлы (*/assets*):

![ScreenShot](screenshots/4.png)

Перейдем в найденную директорию:

![ScreenShot](screenshots/5.png)

В итоге особо ничего интересного в этой директории нет. Тогда попробуем найти другие директории по самому обычному словарю:
```sh
gobuster dir -u http://10.10.12.154/ -w /usr/share/wordlists/dirb/common.txt
```
![ScreenShot](screenshots/6.png)

Перейдем в *robots.txt*:

![ScreenShot](screenshots/7.png)

Находим какую-то странную строку, но ведь это может являться паролем. Соответственно, есть логин и пароль, попробуем подключиться по SSH:

![ScreenShot](screenshots/8.png)

Дальше мне пришлось достаточно долго искать директории через разные базовые словари. В итоге были найдены две новые директории: */login.php* и */portal.php*
```sh
gobuster dir -u http://10.10.12.154/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
```
![ScreenShot](screenshots/9.png)

Переходим на */login.php*:

![ScreenShot](screenshots/10.png)

Попробуем авторизоваться по найденным ранее логину и паролю, и нам это удается!

![ScreenShot](screenshots/11.png)

Видим командную панель. В таком случае пробуем ввести какую-нибудь базовую команду, например, посмотрим файлы в текущей директории:

![ScreenShot](screenshots/12.png)

Видим один из ингридиентов, попробуем прочитать файл:

![ScreenShot](screenshots/13.png)

К сожалению, команда заблокирована. Посмотрим, от какого лица мы находимся в системе:

![ScreenShot](screenshots/14.png)

Мы - **www-data**. Посмотрим, какие команды мы можем выполнять от лица sudo:

![ScreenShot](screenshots/15.png)

На удивление мы можем выполнять абсолютно все команды от лица суперпользователя. Проверим, есть ли доступ к python или python3:

![ScreenShot](screenshots/16.png)
![ScreenShot](screenshots/17.png)

Отлично! Команды исполняются. В таком случае попытаемся получить reverse shell через python3. Для начала на основной системе включаем прослушивание порта, возьмем порт 4444:

![ScreenShot](screenshots/18.png)

Готовые решения можно найти в популярном репозитории PayloadsAllTheThing (https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md). В шаблон вставляем нужный порт (у нас это 4444) и tun0 IP-адрес:

![ScreenShot](screenshots/19.png)

Применяем команду и получаем желаемое на 4444 порте:

![ScreenShot](screenshots/20.png)

### Question 1: What is the first ingredient Rick needs?
Посмотрим на файлы и прочитаем файл *clue.txt*:

![ScreenShot](screenshots/21.png)

Смотрим файл *Sup3rS3cretPickl3Ingred.txt*:

![ScreenShot](screenshots/22.png)

Первый ингредиент - **mr. meeseek hair**

### Question 2: Whats the second ingredient Rick needs?
Поднимемся в корневую директори и далее перейдем в */home*. Видим двух пользователей - **rick** и **ubuntu**. Посмотрим в папку пользователя *rick*:

![ScreenShot](screenshots/23.png)

Видим файл со вторым ингредиентом. Читаем его:

![ScreenShot](screenshots/24.png)

Второй ингредиент - **1 jerry tear**

### Question 3: Whats the final ingredient Rick needs?
В папку */root* просто так перейти нельзя:

![ScreenShot](screenshots/25.png)

Но мы можем посмотреть внутрь нее при помощи sudo, а также прочитать этот файл тем же методом:

![ScreenShot](screenshots/26.png)

Третий ингредиент - **fleeb juice**

P.S. посмотрел другие способы решения и увидел, что все три ингредиента можно получить не получая reverse shell, например при помощи команды *less* или экранируя какой-либо символ в команде. И да, в исходном коде страницы можно было посмотреть список заблокированных команд:

![ScreenShot](screenshots/27.png)
