# TryHackMe: Valley

## Task 1: Get those flags!
Используем Nmap для сканирования машины:
```sh
nmap -sC -sV -p- 10.10.150.67
```
![ScreenShot](screenshots/1.png)

Мы нашли:
- 22 port - SSH (OpenSSH 8.2p1)
- 80 port - HTTP (Apache httpd 2.4.41)
- 37370 port - FTP (vsftpd 3.0.3)

Первым делом, перейдем на сайт и осмотрим его:

![ScreenShot](screenshots/2.png)

![ScreenShot](screenshots/3.png)

![ScreenShot](screenshots/4.png)

![ScreenShot](screenshots/5.png)

Осмотрев сайт, находим следующие директории:
- /
- /pricing
- /gallery
- /static (и вложеные директории по номерам фотографий)

В директории **/pricing** присутствует файл **note.txt**:

![ScreenShot](screenshots/6.png)

![ScreenShot](screenshots/7.png)

В записке рекомендуется некому пользователю **J** прекратить размещать записи на веб-ресурсе. Возможно, мы можем найти и другие записи. Попробуем посканировать сайт на наличие других директорий:

![ScreenShot](screenshots/8.png)

Сканирование относительно корневой директории не дало новых результатов в отличие от сканирования относительно директории **/static**:

![ScreenShot](screenshots/9.png)

Найдена интересная директория - **/00**:

![ScreenShot](screenshots/10.png)

В найденной директории располагается еще одна запись, в которой мы находим директорию **/dev1243224123123**:

![ScreenShot](screenshots/11.png)

Обнаруживаем страницу логина, а в исходном коде страницы находим и данные для входа (**siemDev:california**): 

![ScreenShot](screenshots/12.png)

Проходим авторизацию при помощи найденных логина и пароля и попадаем на новую запись:

![ScreenShot](screenshots/13.png)

Обращаем внимание на пункт **stop reusing credentials**, а также на заголовок **dev notes for ftp server**. Пробуем войти на FTP-сервер, используя найденные ранее логин и пароль:

![ScreenShot](screenshots/14.png)

Скачиваем все найденные файлы:

![ScreenShot](screenshots/15.png)

Проверяем скачанные дампы сетевого трафика по порядку, начиная с файла **siemFTP.pcapng**:

![ScreenShot](screenshots/16.png)

В этом файле находим лишь упоминание про вход под анонимным пользователем на FTP-сервер. Собственно, на данный момент эта возможность отключена:

![ScreenShot](screenshots/17.png)

Исследуем файл **siemHTTP1.pcapng**:

![ScreenShot](screenshots/18.png)

Через **экспорт HTTP-объектов** находим ресурсы, с которыми были произведены какие-либо манипуляции со стороны пользователя. Присутствует взаимодействие с необычным сайтом **www.testingmcafeesites.com**, но среди HTTP-объектов, связанных с этим сайтом, ничего интересного найдено не было:

![ScreenShot](screenshots/19.png)

В третьем файле (**siemHTTP2.pcapng**) находим другой интересный ресурс - **192.168.111.136**

![ScreenShot](screenshots/20.png)

Среди HTTP-объектов, связанных с этим ресурсом, находим объект, в котором в открытом виде лежат логин и пароль - **valleyDev:ph0t0s1234**

![ScreenShot](screenshots/21.png)

Попробуем подключится по SSH, учитывая то, что пользователь любит повторять логины и пароли:

![ScreenShot](screenshots/22.png)

У нас это выходит и получаем первый флаг:

![ScreenShot](screenshots/23.png)

### Question 1: What is the user flag? - THM{k@l1_1n_th3_v@lley}

Проверяем доступность к sudo:

![ScreenShot](screenshots/24.png)

Запуск команд с sudo нам недоступен. В таком случае попробуем найти SUID-файлы:

```sh
find / -perm -4000 2>/dev/null
```

![ScreenShot](screenshots/25.png)

Попробовав различные базовые варианты повышения привилегий с найденными командами, не получаем какого-либо результата. Проверим, есть ли какие-то другие пользователи в системе:

![ScreenShot](screenshots/26.png)

Находим не только пользователей, но и файл **valleyAuthenticator**. Скопируем этот файл к себе на машину:

![ScreenShot](screenshots/27.png)

![ScreenShot](screenshots/28.png)

Находим хэш пароля и через **CrackStation** получаем сам пароль - **liberty123**:

![ScreenShot](screenshots/29.png)

![ScreenShot](screenshots/30.png)

Запускаем файл для проверки логина (логин найден в директории **/home**) и пароля:

![ScreenShot](screenshots/31.png)

В итоге, данные для входа следующие - **valley:liberty123**

Осуществляем переход в другую УЗ:

![ScreenShot](screenshots/32.png)

Находим файл с расширением .py (**photosEncrypt.py**) в **crontab**:

![ScreenShot](screenshots/33.png)

Внутри найденного скрипта используется библиотека **base64**. Осуществим переходм в файл с данной библиотекой и пропишем команду, которая позволит нам (пользователю **valley**) исполнить файл **photosEncrypt.py** с разрешениями фладельца файла, а в качестве владельца выступает пользователь **root**:

```sh
os.system('chmod +s /bin/bash')
```

![ScreenShot](screenshots/34.png)

Иными словами мы разрешаем пользователю **valley** исполнить команду **bash** с разрешениями root-пользователя.

Выполним команду **bash -p** и получим доступ от лица root'а:

![ScreenShot](screenshots/35.png)

Остается только прочитать второй флаг:

![ScreenShot](screenshots/36.png)

### Question 2: What is the root flag? - THM{v@lley_0f_th3_sh@d0w_0f_pr1v3sc}
