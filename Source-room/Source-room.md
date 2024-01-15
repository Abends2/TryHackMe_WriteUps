# TryHackMe: Source

---

## Task 1: Embark

Используем Nmap для сканирования машины:

```sh
sudo nmap -sC -sV 10.10.58.213
```

![ScreenShot](screenshots/1.png)

Мы нашли:
- 22 port - SSH (OpenSSH 7.6p1)
- 10000 port - HTTP (MiniServ 1.890)

Панель Webmin:

![ScreenShot](screenshots/2.png)

За неимением данных для входа, посмотрим существующие эксплойты в Metasploit:

![ScreenShot](screenshots/3.png)

Нас интересует Backdoor (CVE-2019-15107) (RCE)

![ScreenShot](screenshots/4.png)

Посмотрим опции:

![ScreenShot](screenshots/5.png)

Реализуем атаку:

![ScreenShot](screenshots/6.png)

Как итог, получаем доступ к системе от root. Остается только прочитать флаги:

![ScreenShot](screenshots/7.png)

### Question 1: user.txt - THM{SUPPLY_CHAIN_COMPROMISE}

### Question 2: root.txt - THM{UPDATE_YOUR_INSTALL}

---
