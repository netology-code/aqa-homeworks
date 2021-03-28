# Инструкция по установке Docker

1. Зарегистрироваться (получить Docker ID) на [Docker Hub](https://hub.docker.com/).

Выбираете `Sign Up`:

![](pic/signup.png)

Заполняете форму, регистрируйтесь.

2. Определиться с вашей ОС и версией:
* Пользователи Windows 8, 7 - вам нужен Docker Toolbox. Скачать [здесь](https://github.com/docker/toolbox/releases)
* Пользователи Windows 10 - вам нужен Docker Desktop. Установка описана [здесь](https://docs.docker.com/docker-for-windows/install/).
* Пользователи MacOS (год выпуска 2010+ и ОС 10.13 и выше) - вам нужен Docker Desktop. Установка описана [здесь](https://docs.docker.com/docker-for-mac/install/)
* Пользователи Linux, в зависимости от дистрибутива: [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/), [Debian](https://docs.docker.com/install/linux/docker-ce/debian/). Не забудьте так же про [Post Installation](https://docs.docker.com/install/linux/linux-postinstall/)

**Важно**: замечание для пользователей Docker Toolbox на Windows - вам вместо localhost придётся писать `192.168.99.100`. Это ip виртуальной linux машины, в которой уже и запускается docker.

Если работать не будет, то выполните в консоли команду `docker-machine ip default` и увидите адрес (его нужно будет использовать во всех примерах вместо `localhost`).

Q: Что делать, если ничего не получилось?
A: Обратитесь за консультацией к координатору группы или к преподавателю.

## Работа с Play With Docker

Логинитесь, получаете сессию в несколько часов:

![](pic/play.png)

Нажимаете `ADD NEW INSTANCE`, чтобы получить консоль:

![](pic/console.png)

Как закинуть туда файлы:
1. Там есть Git, поэтому можете просто выложить себе в репо нужные файлы и склонировать (самый простой вариант)
2. Есть wget, вы можете выбирать адреса файлов с GitHub и использовать для скачивания:

![](pic/wget.png)

Java установленной там нет, но это не проблема.

Заходите на сайт https://jdk.java.net/13/ и копируете в буфер обмена ссылку на Linux/x64:

![](pic/jdk.png)

Дальше можете скачать через wget и распаковать.

Упрощённой альтернативой является установка через apt:
```
apk add openjdk8

java -version
```

Вы должны увидеть:
```
openjdk version "1.8.0_222"
OpenJDK Runtime Environment (IcedTea 3.13.0) (Alpine 8.222.10-r0)
OpenJDK 64-Bit Server VM (build 25.222-b10, mixed mode)
```

Далее вам немного нужно будет познакомиться с консольным менеджером tmux, который позволяет вам в одной консоли эмулировать несколько:
```
tmux
```

[Документация по tmux](http://xgu.ru/wiki/tmux)

Вам нужно только вот эти горячие клавиши:
* Создание нового окна: ctrl + b + c
* Переход на следующее окно (текущее выделено *): ctrl + b + n
* Закрытие текущего окна: ctrl + b + x

Как проверить, что возвращает сервис на запрос GET:
```
curl http://localhost:9999
```

Редактировать файлы `Dockerfile` и `docker-compose.yml` вы можете как прямо в терминале (но тогда вам нужно использовать nano или vim), либо прямо на GitHub'е в режиме редактирования (тогда просто в Playground делаете `git pull` после каждого сохранения).
