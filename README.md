# Домашнее задание к занятию 8.3 «GitLab»

### Инструкция по выполнению домашнего задания

1. Сделайте fork [репозитория c Шаблоном решения](https://github.com/netology-code/sys-pattern-homework) к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/gitlab-hw или https://github.com/имя-вашего-репозитория/8-03-hw).
2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
   - впишите вверху название занятия и вашу фамилию и имя
   - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
   - для корректного добавления скриншотов воспользуйтесь инструкцией ["Как вставить скриншот в шаблон с решением"](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
   - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.

Желаем успехов в выполнении домашнего задания!

---

### Задание 1

**Что нужно сделать:**

1. Разверните GitLab локально, используя Vagrantfile и инструкцию, описанные в [этом репозитории](https://github.com/netology-code/sdvps-materials/tree/main/gitlab).   
2. Создайте новый проект и пустой репозиторий в нём.
3. Зарегистрируйте gitlab-runner для этого проекта и запустите его в режиме Docker. Раннер можно регистрировать и запускать на той же виртуальной машине, на которой запущен GitLab.

В качестве ответа в репозиторий шаблона с решением добавьте скриншоты с настройками раннера в проекте.

Ответ:

Последовательность выполнения работы.

Открываем vagrantfile и последовательно выполняем следующие установки:

```
sudo apt update
# install docker & docker-compose
sudo apt install -y docker.io docker-compose
# install gitlab: https://about.gitlab.com/install/#ubuntu
sudo apt install -y curl openssh-server ca-certificates tzdata perl
```

Тут взял источник gitlab-ce вместо gitlab-ee. Причина: gitlab-ce - с открытым исходным кодом, установка прошла без дополнительных запросов о лицензии. С gitlab-ee с ходу не заладилось как-то ))
```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo EXTERNAL_URL="http://gitlab.localdomain" apt-get install gitlab-ce
```
Итог получаем:

![screen1](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen1.jpg)

На заметку, для первого входа в наш локальный GitLab создается аккаунт админа по умолчанию:
```
Username: root
Password: fn2yrFJ084+hB6RxTlBY/vgw/p7sJDjpk72z541q0nI=
Пароль у всех конечно свой и лежит тут: /etc/gitlab/initial_root_password
```

Продолжаем добавлять необходимые компоненты из vagrantfile:

```
# pull some images in advance
docker pull gitlab/gitlab-runner:latest
docker pull sonarsource/sonar-scanner-cli:latest
docker pull golang:1.17
docker pull docker:latest
```

Перехожу к запуску установленного GitLab (IP моего хоста - 192.168.1.221):

![screen2](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen2.jpg)
![screen3](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen3.jpg)

После входа на вебку GitLab советую поменять пароль на свой, дабы ранее было такое предупреждение:

Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.

Создаю новый проект и пустой репозиторий в нём. Проект и будет нашим репозиторием.
Выбираем в меню Projects - New project - Create blank project

![screen4](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen4.jpg)
![screen5](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen5.jpg)

Даем название нашему репозиторию (проекту) – «my_first_project_gitlab», делаю его публичным и убираю галочку с создания файла README, чтобы репозиторий оставался чистым, без комитов.

![screen6](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen6.jpg)
![screen7](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen7.jpg)

Регистрируем gitlab-runner для этого проекта и запускаем его в режиме Docker

Раннеры можно увидеть в Settings - CI/CD - Runners

![screen8](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen8.jpg)

Здесь из полезного: ссылка на наш проект (куда подключаться http://192.168.1.221/) и токен: GR1348941-tUzvQz_85itmMCz61m8

Регистрируем наш раннер:

```
docker run -ti --rm --name gitlab-runner \
     --network host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest register
```

![screen9](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen9.jpg)

```
nano /srv/gitlab-runner/config/config.toml
это то что у нас получилось – наша конфигурация
Добавим в нее:
volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"] – даем раннеру доступ к сокету докера

Запускаем раннер:

docker run -d --name gitlab-runner --restart always \
     --network host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest

```
![screen10](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen10.jpg)
![screen11](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen11.jpg)

Как итог теперь в GitLab отобразится наш запущенный раннер:

![screen12](https://github.com/KorolkovDenis/8.3-GitLab/blob/main/Screenshots/task1/screen12.jpg)


---

### Задание 2

**Что нужно сделать:**

1. Запушьте [репозиторий](https://github.com/netology-code/sdvps-materials/tree/main/gitlab) на GitLab, изменив origin. Это изучалось на занятии по Git.
2. Создайте .gitlab-ci.yml, описав в нём все необходимые, на ваш взгляд, этапы.

В качестве ответа в шаблон с решением добавьте: 
   
 * файл gitlab-ci.yml для своего проекта или вставьте код в соответствующее поле в шаблоне; 
 * скриншоты с успешно собранными сборками.
 
 Ответ:

Последовательность выполнения работы.

![screen1](https://github.com/KorolkovDenis/)
![screen1](https://github.com/KorolkovDenis/)
![screen1](https://github.com/KorolkovDenis/)
![screen1](https://github.com/KorolkovDenis/)

 
 
---
## Дополнительные задания* (со звёздочкой)

Их выполнение необязательное и не влияет на получение зачёта по домашнему заданию. Можете их решить, если хотите лучше разобраться в материале.

---

### Задание 3*

Измените CI так, чтобы:

 - этап сборки запускался сразу, не дожидаясь результатов тестов;
 - тесты запускались только при изменении файлов с расширением *.go.

В качестве ответа добавьте в шаблон с решением файл gitlab-ci.yml своего проекта или вставьте код в соответсвующее поле в шаблоне.

![screen1](https://github.com/KorolkovDenis/)


 ## Моя работа в Google:

[Моя работа по GitLab](https://docs.google.com/)
