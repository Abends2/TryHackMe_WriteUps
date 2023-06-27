# TryHackMe: Lian_Yu

## Task 1: Find the Flags

### Question 1: Deploy the VM and Start the Enumeration - :heavy\_check_mark:

Для начала проведем сканирование целевого хоста при помощи Nmap:
```sh
nmap -sC -sV 10.10.48.198 -T5
```

![ScreenShot](screenshots/1.png)

Найденные сервисы и службы:
- 21 port - FTP (vsftpd 3.0.2)
- 22 port - SSH (OpenSSH 6.7p1)
- 80 port - HTTP (Apache httpd)
- 111 port - RPC (rpcbind 2-4)

Посмотрим на сайт, который расположен на 80 порте:

![ScreenShot](screenshots/2.png)

Запустим сканирование директорий сайта:
```sh
gobuster dir -u http://10.10.48.198 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![ScreenShot](screenshots/3.png)

Находим директорию **/island**:

![ScreenShot](screenshots/4.png)

Среди текста на странице, находим скрытое ключевое слово **vigilante**:

![ScreenShot](screenshots/5.png)

Снова запускам Gobuster, но перебор будем осуществлять внутри директории **/island**:
```sh
gobuster dir -u http://10.10.48.198/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

![ScreenShot](screenshots/6.png)

Находим директорию **/2100**:

![ScreenShot](screenshots/7.png)

### Question 2: What is the Web Directory you found? - /2100

Самое интересное, что видео с YouTube заблокировано **:)**

В прочем, это, забегая вперед, не играет никакой роли. В исходном коде страницы была найдена подсказка **.ticket**. 

![ScreenShot](screenshots/8.png)

Попробуем найти файл с таким расширением на сайте (/island/2100/).
```sh
gobuster dir -u http://10.10.48.198/island/2100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .ticket
```

![ScreenShot](screenshots/9.png)

В итоге находим следующий файл: **green_arrow.ticket**

### Question 3: What is the file name you found? - green_arrow.ticket

Перейдем к самому файлу:

![ScreenShot](screenshots/10.png)

Находим некий "токен" (по всей видимости, пароль) - **RTy8yhBQdscX**. Вот только этот пароль не подходит для FTP, по всей видимости он зашифрован.

При помощи инструмента CyberChef и компонента Base58 расшифровываем пароль:

![ScreenShot](screenshots/11.png)

### Question 4: What is the FTP Password? - !#th3h00d

Перейдем на FTP-сервер при помощи найденных логина и пароля: **vigilante**:**!#th3h00d**. Файлы, находящиеся в директории **/home/vigilante**:

![ScreenShot](screenshots/12.png)

Скачиваем все найденные картинки:

![ScreenShot](screenshots/13.png)

![ScreenShot](screenshots/14.png)

Замечаем, что один из файлов поврежден, проверим его на наличие, каких-либо артефактов или данных.

![ScreenShot](screenshots/15.png)

![ScreenShot](screenshots/16.png)

Видим, что файл, скорее всего, поломан. Посмотрим его в hex-редакторе (hexedit).
```sh
hexedit Leave_me_alone.png
```

![ScreenShot](screenshots/17.png)

Блоки IDAT и IHDR подтверждают структуру PNG-файла, но, по всей видимости, он поломан именно в самом начале. Воспользуемся памяткой с правильной структурой PNG.

![ScreenShot](screenshots/18.png)

Поменяем значения некоторых байт в соотвествии с шаблоном:

![ScreenShot](screenshots/19.png)

В итоге, имеем восстановленную картинку:

![ScreenShot](screenshots/20.png)

Получаем только **password**. Возможно, с помощью **steghide** в изображении что-то скрыто, а **password** является парольной фразой:

![ScreenShot](screenshots/21.png)

Хм, в самой картинке ничего нет, попробуем то же самое с другими:

![ScreenShot](screenshots/22.png)

Извлекли архив. Внутри него два файла:

![ScreenShot](screenshots/23.png)

В файле **shado** находится пароль от SSH - **M3tahuman**

![ScreenShot](screenshots/24.png)

### Question 5: What is the file name with SSH password - M3tahuman

Во втором файле находится какое-то общее описание:

![ScreenShot](screenshots/25.png)

Но вот у нас нет логина для подключения по SSH. Тут мы обращаем еще раз внимание на содержимое FTP-сервера. В папке **/vigilante** находится файл **.other_user**, в котором речь идет о неком **Slade Wilson**. 

![ScreenShot](screenshots/26.png)

Пробуем в качестве логина **slade**, а в качестве пароля **M3tahuman**:

![ScreenShot](screenshots/27.png)

И да! Мы внутри. Забираем первый флаг:

![ScreenShot](screenshots/28.png)

### Question 6: user.txt - THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}

Посмотрим, что мы можем выполнять в системе от лица sudo:

![ScreenShot](screenshots/29.png)

Нам доступна команда pkexec. Можно ли через нее повысить привилегии? Можно **:)**

![ScreenShot](screenshots/30.png)

![ScreenShot](screenshots/31.png)

Остается дело за малым, прочитать root-флаг:

![ScreenShot](screenshots/32.png)

### Question 7: root.txt - THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
