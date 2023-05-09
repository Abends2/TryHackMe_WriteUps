# TryHackMe: Mr Robot CTF


## Task 1: Hack the machine
Используем nmap для сканирования машины:
```sh
nmap -sC -sV 10.10.171.170
```
![ScreenShot](screenshots/1.png)

Мы нашли:
- 22 port - SSH (closed)
- 80 port - HTTP (Apache httpd)
- 443 port - HTTPS (Apache httpd)

Перейдем на сам сайт (80 порт):

![ScreenShot](screenshots/2.png)

На выбор предоставляются некоторые команды, но, проверив их, ничего интересного они не приносят в плане развития атаки. Вот некоторые примеры исполнения "команд":

![ScreenShot](screenshots/3.png)

![ScreenShot](screenshots/4.png)

![ScreenShot](screenshots/5.png)

![ScreenShot](screenshots/6.png)

На последней картинке в исходном коде страницы можно увидеть интересный баннер: **"YOU ARE NOT ALONE"**

Далее попробуем найти какие-либо директории на сайте:

![ScreenShot](screenshots/7.png)

Директория **/readme**:

![ScreenShot](screenshots/8.png)

Директория **/0**:

![ScreenShot](screenshots/9.png)

Из интресного, был найден файл **robots.txt**:

![ScreenShot](screenshots/10.png)

Здесь можно увидеть 2 скрытые директории (точнее файла). В первой находится первый флаг:

![ScreenShot](screenshots/11.png)

### Question 1: What is key 1? - 073403c8a58a1f80d943455fb30724b9

А в другой находится словарь, который нам скорее всего пригодится далее. После этого, обратим внимание на другие найденные директории с обозначением **wp**, что говорит нам о том, что в основе сайта лежит **WordPress**:

![ScreenShot](screenshots/12.png)

Далее перехватим запрос при помощи **BurpSuite** при попытке авторизации на сайте:

![ScreenShot](screenshots/13.png)

Словарь, который мы нашли ранее, похож на словарь с логинами (а может быть и паролями):

![ScreenShot](screenshots/14.png)

Перенаправим захваченный запрос в Intruder, выделив поле **login** для подстановки значений из найденного словаря:

![ScreenShot](screenshots/15.png)

![ScreenShot](screenshots/16.png)

По длине ответа находим логин **Elliot**:

![ScreenShot](screenshots/17.png)

Теперь попробуем найти пароль при помощи инструмента **WPScan**:

![ScreenShot](screenshots/18.png)

![ScreenShot](screenshots/19.png)

Найденный пароль - **ER28-0652** (операция по нахождению пароля действительно долгая, потому что пароль оказался внизу списка). Входим в аккаунт:

![ScreenShot](screenshots/20.png)

Посмотрим вкладку Users:

![ScreenShot](screenshots/21.png)

Узнаем, что Elliot - администратор, а также помимо него есть и другой пользователь. В одной из тем находим интересный шаблон с расширением **.php**.

![ScreenShot](screenshots/22.png)

Исходный код найденного шаблона:

![ScreenShot](screenshots/23.png)

Скорее всего, таким способом можно реализовать reverse shell, заменив содержимое шаблона на код из файла phpreverseshell от pentestmonkey, но перед этим реализуем прослушивание порта на хостовой системе:

![ScreenShot](screenshots/24.png)

К слову, здесь reverse shell можно получить несколькими способами, например просто вызвав нужную php-функцию:

![ScreenShot](screenshots/25.png)

Переходим непосредственно к этому шаблону, чтобы активировать скрипт, но перед этим не забываем сохранить изменения:

![ScreenShot](screenshots/26.png)

В итоге у нас получилось реализовать reverse shell:

![ScreenShot](screenshots/27.png)

Забегая вперед, скажу, что данный метод у меня впоследствии не сработал в плане дальнейшего развития атаки - я не смог перейти к другому пользователю из-за предупреждения - **su: must be run from a terminal**

Находим директорию со вторым ключом, но файл прочитать не можем:

![ScreenShot](screenshots/28.png)

Далее я реализовал reverse shell иным способом:

![ScreenShot](screenshots/29.png)

Активируем:

![ScreenShot](screenshots/30.png)

И получаем новый shell:

![ScreenShot](screenshots/31.png)

Вызываем bash-оболочку:

![ScreenShot](screenshots/32.png)

Продолжаем работу. Помимо второго ключа был найден и файл с хэшем пароля. Попробуем найти этот пароль через **CrackStation**:

![ScreenShot](screenshots/33.png)

![ScreenShot](screenshots/34.png)

Пароль - **abcdefghijklmnopqrstuvwxyz**

Вот и та проблема, которая у меня возникла:

![ScreenShot](screenshots/35.png)

Тут, как уже говорилось ранее, был реализован новый reverse shell. При этом входим относительно пользователя robot:

![ScreenShot](screenshots/36.png)

Получаем второй флаг:

![ScreenShot](screenshots/37.png)

### Question 2: What is key 2? - 822c73956184f694993bede3eb39f959

Проверим наши права:

![ScreenShot](screenshots/38.png)

Так, пользователь не может использовать sudo, поэтому попробуем по-другому:

![ScreenShot](screenshots/39.png)

Из интересного, нам доступен **nmap**. Проверим, можно ли через него повысить привилегии:

![ScreenShot](screenshots/40.png)

![ScreenShot](screenshots/41.png)

Да, действительно можно, надо просто запустить nmap в интерактивном режиме:

![ScreenShot](screenshots/42.png)

Вот мы и получили root'а, получаем третий флаг:

![ScreenShot](screenshots/43.png)

### Question 3: What is key 3? - 04787ddef27c3dee1ee161b21670b4e4
