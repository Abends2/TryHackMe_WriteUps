# TryHackMe: Easy Peasy

## Task 1: Enumeration through Nmap

Используем Nmap для сканирования машины:
```sh
sudo nmap -sC -sV -p- -Pn 10.10.224.208
```
![ScreenShot](screenshots/1.png)

```sh
nmap -sC -sV -p 65524 10.10.224.208
```
![ScreenShot](screenshots/2.png)

Мы нашли:
- 80 port - HTTP (Nginx 1.16.1)
- 6498 port - SSH (OpenSSH 7.6p1)
- 65524 port - HTTP (Apache 2.4.43)

### Question 1: How many ports are open? - 3

### Question 2: What is the version of nginx? - 1.16.1

### Question 3: What is running on the highest port? - Apache


## Task 2: Compromising the machine

Приверим активные веб-сервисы:

![ScreenShot](screenshots/3.png)

![ScreenShot](screenshots/4.png)

Обнаруживаем стандартные страницы Apache и Nginx. Помимо этого, к заданию приложен файл **easypeasy.txt**, который представляет собой словарь:

![ScreenShot](screenshots/5.png)

Используем **gobuster** для поиска директорий на сайтах:
```sh
gobuster dir -u http://10.10.224.208:80 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![ScreenShot](screenshots/6.png)

Находим директорию **/hidden**, но тут совершенно пусто:

![ScreenShot](screenshots/7.png)

Ищем дальше:

```sh
gobuster dir -u http://10.10.224.208:80/hidden -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![ScreenShot](screenshots/8.png)

Внутри директории **/hidden** находим директорию **/whatever**:

![ScreenShot](screenshots/9.png)

Внутри обнаруживаем скрытую строку в формате base64, это и есть первый флаг:

![ScreenShot](screenshots/10.png)

### Question 4: Using GoBuster, find flag 1 - flag{f1rs7_fl4g}

Далее смотрим в сторону **robots.txt** на другом веб-ресурсе:

![ScreenShot](screenshots/11.png)

Тут в поле **User-Agent** находим хэш, это наш второй флаг:

![ScreenShot](screenshots/12.png)

### Question 5: Further enumerate the machine, what is flag 2? - flag{1m_s3c0nd_fl4g}

Затем был найден третий флаг, который скрыт среди текста в **Apache Default Page**:

![ScreenShot](screenshots/13.png)

На **CrackStation** находим значение хэша - **candeger** -  внутри обертки флага:

![ScreenShot](screenshots/14.png)

Странно, но в качестве ответа принимается именно хэш, обернутый во flag{}, не могу сказать, почему именно так происходит, но факт есть факт.

### Question 6: Crack the hash with easypeasy.txt. What is the flag 3? - flag{9fdafbd64c47471a8f54cd3fc64cd312}

Также внутри **Apache Default Page** находим скрытый зашифрованный текст:

![ScreenShot](screenshots/15.png)

Через идентификатор шифра узнаем, что это Base62:

![ScreenShot](screenshots/16.png)

Тем самым, мы нашли скрытую директорию.

### Question 7: What is the hidden directory? - /n0th1ng3ls3m4tt3r

Если перейти в скрытую директорию, можно обнаружить картинку:

![ScreenShot](screenshots/17.png)

А также и скрытый текст, который представляет собой хэш:

![ScreenShot](screenshots/18.png)

Найденный хэш расшифровываем при помощи инструмента **JohnTheReaper** и словаря **easypeasy.txt**:

![ScreenShot](screenshots/19.png)

### Question 8: Using the wordlist that provided to you in this task crack the hash. What is the password? - mypasswordforthatjob

Теперь разбираемся с картинкой. Во-первых, скачиваем ее себе:

![ScreenShot](screenshots/20.png)

Во-вторых, с помощью **steghide** и пароля **mypasswordforthatjob** получаем текстовый файл, который был спрятан внутри картинки:

![ScreenShot](screenshots/21.png)

Внутри обнаруживаем логин **boring** и пароль в двочном представлении. Переводим его в текстовый вариант:

![ScreenShot](screenshots/22.png)

![ScreenShot](screenshots/23.png)

### Question 9: What is the password to login to the machine via SSH? - iconvertedmypasswordtobinary

Подключаемся по SSH с помощью найденных данных и читаем первый флаг:

![ScreenShot](screenshots/24.png)

![ScreenShot](screenshots/25.png)

Флаг оказывается зашифрованным, но это всего лишь ROT13:

![ScreenShot](screenshots/26.png)

### Question 10: What is the user flag? - flag{n0wits33msn0rm4l}

Проверим, можем ли мы использовать sudo:

![ScreenShot](screenshots/27.png)

К сожалению, не можем. Посмотрим в **crontab**:

![ScreenShot](screenshots/28.png)

Здесь обнаруживаем, что относительно пользователя **root** исполняется bash-скрипт **.mysecretcronjob.sh**:

![ScreenShot](screenshots/29.png)

Внутри этого скрипта нет полезной нагрузки. Только подсказка, которая говорит нам о том, что все команды внутри данного скрипта выполняются относительно суперпользователя. Получается, что, реализовав reverse shell через этот скрипт, мы получим доступ к root-пользователю. Проверим версию python3:

![ScreenShot](screenshots/30.png)

Находим команду, которая поможет реализовать reverse shell:

![ScreenShot](screenshots/31.png)

Переносим полученную команду внутрь скрипта:

![ScreenShot](screenshots/32.png)

Не забывает поставить порт на прослушивание:

![ScreenShot](screenshots/33.png)

Активируем скрипт и получаем полный доступ к системе:

![ScreenShot](screenshots/34.png)

Читаем последний флаг:

![ScreenShot](screenshots/35.png)

### Question 11: What is the root flag? - flag{63a9f0ea7bb98050796b649e85481845}
