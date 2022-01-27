
# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"

## Задача 1

Сценарий выполения задачи:

- создайте свой репозиторий на https://hub.docker.com;
- выберете любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.  

Ответ:  
<details>
     <summary>Решение Задачи 1</summary>
    <br>

 - Зарегистрировался на hub.docker.com
 - Образ буду использовать nginx:stable
 - pull'им образ
```bash
[gnoy@manjarokde-ws01 task1]$ docker pull nginx:stable
stable: Pulling from library/nginx
a2abf6c4d29d: Pull complete 
da03644a1293: Pull complete 
dcbfc6badd70: Pull complete 
3f7ccff97047: Pull complete 
49e31097680b: Pull complete 
c423e1dacb26: Pull complete 
Digest: sha256:8e6a9791a85b9583de5ae22d0ade6d1bb36aab21708259441d230d12907e2dcb
Status: Downloaded newer image for nginx:stable
docker.io/library/nginx:stable
```
 - Подготавливаю index.html (из задания), создаю Dockerfile для создания образа
```Dockerfile
FROM nginx:stable

COPY index.html /usr/share/nginx/html
```
```bash
[gnoy@manjarokde-ws01 task1]$ docker build -t nginx-stable:homework1 .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM nginx:stable
 ---> 50fe74b50e0d
Step 2/2 : COPY index.html /usr/share/nginx/html
 ---> bbec3a5ebd5a
Successfully built bbec3a5ebd5a
Successfully tagged nginx-stable:homework1
```
 - Запускаю контейнер, проверяю, что запущен
```bash
[gnoy@manjarokde-ws01 task1]$ docker run --name docker-homework1 -p 80:80 -d nginx-stable:homework1
6b3fcbb76a0eff428c80ba36e88af875b5cc35962528663a001b17e88cf6dd8b
[gnoy@manjarokde-ws01 task1]$ docker ps
CONTAINER ID   IMAGE                    COMMAND                  CREATED         STATUS         PORTS                               NAMES
6b3fcbb76a0e   nginx-stable:homework1   "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   docker-homework1
```
 - Проверяю доступность индекс-страницы
```bash
[gnoy@manjarokde-ws01 task1]$ lynx http://localhost --dump
   Hey, Netology

                              I'm DevOps Engineer!
```
---
### he's alive!! :D
---
 - "пушу" получившейся образ на docker хаб
```bash
[gnoy@manjarokde-ws01 task1]$ docker tag nginx-stable:homework1 gnoy4eg/nginx-stable:homework1
[gnoy@manjarokde-ws01 task1]$ docker push gnoy4eg/nginx-stable:homework1
The push refers to repository [docker.io/gnoy4eg/nginx-stable]
2a0a23a6b28b: Pushed 
c75c795b7d44: Mounted from library/nginx 
4e498ce5ae6a: Mounted from library/nginx 
35437a3771fc: Mounted from library/nginx 
108a6d6c3e60: Mounted from library/nginx 
9ccbab2746b8: Mounted from library/nginx 
2edcec3590a4: Mounted from library/nginx 
homework1: digest: sha256:7982abf4383a72bc72d8e0b21a36693446596ab8e00826a96f3690f69b34a2d2 size: 1777
```
</details>

Ссылка на мой репо:  
https://hub.docker.com/repository/docker/gnoy4eg/nginx-stable

## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.

--

Сценарий:

- Высоконагруженное монолитное java веб-приложение;
- Nodejs веб-приложение;
- Мобильное приложение c версиями для Android и iOS;
- Шина данных на базе Apache Kafka;
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
- Мониторинг-стек на базе Prometheus и Grafana;
- MongoDB, как основное хранилище данных для java-приложения;
- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.  

Ответ:  
- Высоконагруженное монолитное java веб-приложение;  
Докер подойдет если монолит декомпозировать на более мелкие службы\сервисы, но понадобится чуть ли не полная переборка приложения, модификация кода. А пока лучше использовать виртуальную машину т.к. скорость задержек при использовании современного гипервизора не сильно отличается от физической машины, но за счет гипервизора имеем очень много бонусов (снапшоты, бекапирование, клонирование и т.п.).
- Nodejs веб-приложение;  
Докер подойдет, веб-приложение, замечательно будет работать в контейнере, может в последствии обрастать дополнительными "плюшками" на лету.
- Мобильное приложение c версиями для Android и iOS;  
Имеется ввиду файлы .deb и .ipa? Подойдет обычная виртуалка. Постоянно пересобирать контейнер для добавления очередной версии файла (приложения) не имеет смысла, можно просто хранить их на "шаре" в обычной виртуалке.
- Шина данных на базе Apache Kafka;  
Используется данное решение в крупных проектах совместно с Big Data, для данных важна критичность доставки. Докер в паре с оркестратором может обеспечить масштабируемость системы, а следовательно он подходит.
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;  
Стек ELK. Есть решения и в докере и на виртуалках. Если отвечать на вопрос: подходит ли сценарий для использования докер контейнеров? Да, подходит, использовать можно. В рамках курса DevOps-инженер я бы использовал "docker-версию" :)
- Мониторинг-стек на базе Prometheus и Grafana;  
Данный софт не хранит данные. Можно использовать в докере. Имеем масштабируемость и скорость развертывания.
- MongoDB, как основное хранилище данных для java-приложения;  
Докер не подойдет, он запускается на базе неизменяемого образа, а в случае удаления контейнера мы можем потерять все данные. Выбор в пользу виртуалки.
- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.  
Докер не подойдет. Представленные системы хранят много часто изменяемых данных. При удалении контейнера мы можем потерять все данные.


## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.  

Ответ:  
Запускаем контейнеры с centos и debian
```bash
[gnoy@manjarokde-ws01 ~]$ docker run -it -v /home/gnoy/data:/data --name lyuboy_tag -d centos bash
Unable to find image 'centos:latest' locally
latest: Pulling from library/centos
a1d0c7532777: Pull complete 
Digest: sha256:a27fd8080b517143cbbbab9dfb7c8571c40d67d534bbdee55bd6c473f432b177
Status: Downloaded newer image for centos:latest
029e031d2c5d9805503015500f31fd798ae9b8e044dec3e2c52947aa8f6012b9
[gnoy@manjarokde-ws01 ~]$ docker run -it -v /home/gnoy/data:/data -d debian bash
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
0c6b8ff8c37e: Pull complete 
Digest: sha256:fb45fd4e25abe55a656ca69a7bef70e62099b8bb42a279a5e0ea4ae1ab410e0d
Status: Downloaded newer image for debian:latest
1c8ded92634a12c9c9349a091f0efc53e5021c59b85851968668d842808f67f7
[gnoy@manjarokde-ws01 ~]$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED         STATUS         PORTS     NAMES
1c8ded92634a   debian    "bash"    5 minutes ago   Up 5 minutes             nostalgic_villani
029e031d2c5d   centos    "bash"    6 minutes ago   Up 6 minutes             lyuboy_tag
```
Подключаюсь к первому контейнеру (centos) и создаем файл
```bash
[gnoy@manjarokde-ws01 ~]$ docker exec lyuboy_tag bash -c "echo text for centos > /data/centos_file.txt"
```
На хостовой машине создаем файл в папке `/data`, заодно проверим содержимое папки на хостовой машине
```bash
[gnoy@manjarokde-ws01 ~]$ echo "text for file on host machine" > data/host_text_file.txt
[gnoy@manjarokde-ws01 ~]$ ls data
centos_file.txt  host_text_file.txt
```
Подключаюсь к контейнеру с debian и смортю содержимое папки и файлов
```bash
[gnoy@manjarokde-ws01 ~]$ docker exec -it nostalgic_villani bash
root@1c8ded92634a:/# ls data 
centos_file.txt  host_text_file.txt
root@1c8ded92634a:/# cat data/host_text_file.txt 
text for file on host machine
root@1c8ded92634a:/# cat data/centos_file.txt 
text for centos
```

## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.  

Ответ:  
Запускаем:  
```bash
[gnoy@manjarokde-ws01 ansible]$ docker build -t olegbukatchuk/ansible:2.9.24 .
```
<details>
     <summary>Вывод с консоли:</summary>

```bash
[gnoy@manjarokde-ws01 ansible]$ docker build -t olegbukatchuk/ansible:2.9.24 .
Sending build context to Docker daemon   2.56kB
Step 1/5 : FROM alpine:3.14
3.14: Pulling from library/alpine
97518928ae5f: Pull complete 
Digest: sha256:635f0aa53d99017b38d1a0aa5b2082f7812b03e3cdb299103fe77b5c8a07f1d2
Status: Downloaded newer image for alpine:3.14
 ---> 0a97eee8041e
Step 2/5 : RUN CARGO_NET_GIT_FETCH_WITH_CLI=1 &&     apk --no-cache add         sudo         python3        py3-pip         openssl         ca-certificates         sshpass         openssh-client         rsync         git &&     apk --no-cache add --virtual build-dependencies         python3-dev         libffi-dev         musl-dev         gcc         cargo         openssl-dev         libressl-dev         build-base &&     pip install --upgrade pip wheel &&     pip install --upgrade cryptography cffi &&     pip install ansible==2.9.24 &&     pip install mitogen ansible-lint jmespath &&     pip install --upgrade pywinrm &&     apk del build-dependencies &&     rm -rf /var/cache/apk/* &&     rm -rf /root/.cache/pip &&     rm -rf /root/.cargo
 ---> Running in 07940c5f97b4
fetch https://dl-cdn.alpinelinux.org/alpine/v3.14/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.14/community/x86_64/APKINDEX.tar.gz
(1/55) Installing ca-certificates (20211220-r0)
(2/55) Installing brotli-libs (1.0.9-r5)
(3/55) Installing nghttp2-libs (1.43.0-r0)
(4/55) Installing libcurl (7.79.1-r0)
(5/55) Installing expat (2.4.3-r0)
(6/55) Installing pcre2 (10.36-r0)
(7/55) Installing git (2.32.0-r0)
(8/55) Installing openssh-keygen (8.6_p1-r3)
(9/55) Installing ncurses-terminfo-base (6.2_p20210612-r0)
(10/55) Installing ncurses-libs (6.2_p20210612-r0)
(11/55) Installing libedit (20210216.3.1-r0)
(12/55) Installing openssh-client-common (8.6_p1-r3)
(13/55) Installing openssh-client-default (8.6_p1-r3)
(14/55) Installing openssl (1.1.1l-r0)
(15/55) Installing libbz2 (1.0.8-r1)
(16/55) Installing libffi (3.3-r2)
(17/55) Installing gdbm (1.19-r0)
(18/55) Installing xz-libs (5.2.5-r0)
(19/55) Installing libgcc (10.3.1_git20210424-r2)
(20/55) Installing libstdc++ (10.3.1_git20210424-r2)
(21/55) Installing mpdecimal (2.5.1-r1)
(22/55) Installing readline (8.1.0-r0)
(23/55) Installing sqlite-libs (3.35.5-r0)
(24/55) Installing python3 (3.9.5-r2)
(25/55) Installing py3-appdirs (1.4.4-r2)
(26/55) Installing py3-chardet (4.0.0-r2)
(27/55) Installing py3-idna (3.2-r0)
(28/55) Installing py3-urllib3 (1.26.5-r0)
(29/55) Installing py3-certifi (2020.12.5-r1)
(30/55) Installing py3-requests (2.25.1-r4)
(31/55) Installing py3-msgpack (1.0.2-r1)
(32/55) Installing py3-lockfile (0.12.2-r4)
(33/55) Installing py3-cachecontrol (0.12.6-r1)
(34/55) Installing py3-colorama (0.4.4-r1)
(35/55) Installing py3-contextlib2 (0.6.0-r1)
(36/55) Installing py3-distlib (0.3.1-r3)
(37/55) Installing py3-distro (1.5.0-r3)
(38/55) Installing py3-six (1.15.0-r1)
(39/55) Installing py3-webencodings (0.5.1-r4)
(40/55) Installing py3-html5lib (1.1-r1)
(41/55) Installing py3-parsing (2.4.7-r2)
(42/55) Installing py3-packaging (20.9-r1)
(43/55) Installing py3-toml (0.10.2-r2)
(44/55) Installing py3-pep517 (0.10.0-r2)
(45/55) Installing py3-progress (1.5-r2)
(46/55) Installing py3-retrying (1.3.3-r1)
(47/55) Installing py3-ordered-set (4.0.2-r1)
(48/55) Installing py3-setuptools (52.0.0-r3)
(49/55) Installing py3-pip (20.3.4-r1)
(50/55) Installing libacl (2.2.53-r0)
(51/55) Installing popt (1.18-r0)
(52/55) Installing zstd-libs (1.4.9-r1)
(53/55) Installing rsync (3.2.3-r4)
(54/55) Installing sshpass (1.09-r0)
(55/55) Installing sudo (1.9.7_p1-r1)
Executing busybox-1.33.1-r6.trigger
Executing ca-certificates-20211220-r0.trigger
OK: 98 MiB in 69 packages
fetch https://dl-cdn.alpinelinux.org/alpine/v3.14/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.14/community/x86_64/APKINDEX.tar.gz
(1/37) Installing pkgconf (1.7.4-r0)
(2/37) Installing python3-dev (3.9.5-r2)
(3/37) Installing linux-headers (5.10.41-r0)
(4/37) Installing libffi-dev (3.3-r2)
(5/37) Installing musl-dev (1.2.2-r3)
(6/37) Installing binutils (2.35.2-r2)
(7/37) Installing libgomp (10.3.1_git20210424-r2)
(8/37) Installing libatomic (10.3.1_git20210424-r2)
(9/37) Installing libgphobos (10.3.1_git20210424-r2)
(10/37) Installing gmp (6.2.1-r0)
(11/37) Installing isl22 (0.22-r0)
(12/37) Installing mpfr4 (4.1.0-r0)
(13/37) Installing mpc1 (1.2.1-r0)
(14/37) Installing gcc (10.3.1_git20210424-r2)
(15/37) Installing rust-stdlib (1.52.1-r1)
(16/37) Installing libxml2 (2.9.12-r1)
(17/37) Installing llvm11-libs (11.1.0-r2)
(18/37) Installing http-parser (2.9.4-r0)
(19/37) Installing pcre (8.44-r0)
(20/37) Installing libssh2 (1.9.0-r1)
(21/37) Installing libgit2 (1.1.0-r2)
(22/37) Installing rust (1.52.1-r1)
(23/37) Installing cargo (1.52.1-r1)
(24/37) Installing openssl-dev (1.1.1l-r0)
(25/37) Installing libressl3.3-libcrypto (3.3.3-r0)
(26/37) Installing libressl3.3-libssl (3.3.3-r0)
(27/37) Installing libressl3.3-libtls (3.3.3-r0)
(28/37) Installing libressl-dev (3.3.3-r0)
(29/37) Installing libmagic (5.40-r1)
(30/37) Installing file (5.40-r1)
(31/37) Installing libc-dev (0.7.2-r3)
(32/37) Installing g++ (10.3.1_git20210424-r2)
(33/37) Installing make (4.3-r0)
(34/37) Installing fortify-headers (1.1-r1)
(35/37) Installing patch (2.7.6-r7)
(36/37) Installing build-base (0.5-r2)
(37/37) Installing build-dependencies (20220127.065221)
Executing busybox-1.33.1-r6.trigger
OK: 1110 MiB in 106 packages
Requirement already satisfied: pip in /usr/lib/python3.9/site-packages (20.3.4)
Collecting pip
  Downloading pip-21.3.1-py3-none-any.whl (1.7 MB)
Collecting wheel
  Downloading wheel-0.37.1-py2.py3-none-any.whl (35 kB)
Installing collected packages: wheel, pip
  Attempting uninstall: pip
    Found existing installation: pip 20.3.4
    Uninstalling pip-20.3.4:
      Successfully uninstalled pip-20.3.4
Successfully installed pip-21.3.1 wheel-0.37.1
Collecting cryptography
  Downloading cryptography-36.0.1-cp36-abi3-musllinux_1_1_x86_64.whl (3.8 MB)
Collecting cffi
  Downloading cffi-1.15.0.tar.gz (484 kB)
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Collecting pycparser
  Downloading pycparser-2.21-py2.py3-none-any.whl (118 kB)
Building wheels for collected packages: cffi
  Building wheel for cffi (setup.py): started
  Building wheel for cffi (setup.py): finished with status 'done'
  Created wheel for cffi: filename=cffi-1.15.0-cp39-cp39-linux_x86_64.whl size=429204 sha256=aa91e8a394583e16898eb6d7a2a4f9f38f0ebbbfca678b85e6bfbf18ae73a635
  Stored in directory: /root/.cache/pip/wheels/8e/0d/16/77c97b85a9f559c5412c85c129a2bae07c771d31e1beb03c40
Successfully built cffi
Installing collected packages: pycparser, cffi, cryptography
Successfully installed cffi-1.15.0 cryptography-36.0.1 pycparser-2.21
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Collecting ansible==2.9.24
  Downloading ansible-2.9.24.tar.gz (14.3 MB)
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Collecting jinja2
  Downloading Jinja2-3.0.3-py3-none-any.whl (133 kB)
Collecting PyYAML
  Downloading PyYAML-6.0.tar.gz (124 kB)
  Installing build dependencies: started
  Installing build dependencies: finished with status 'done'
  Getting requirements to build wheel: started
  Getting requirements to build wheel: finished with status 'done'
  Preparing metadata (pyproject.toml): started
  Preparing metadata (pyproject.toml): finished with status 'done'
Requirement already satisfied: cryptography in /usr/lib/python3.9/site-packages (from ansible==2.9.24) (36.0.1)
Requirement already satisfied: cffi>=1.12 in /usr/lib/python3.9/site-packages (from cryptography->ansible==2.9.24) (1.15.0)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.0.1-cp39-cp39-musllinux_1_1_x86_64.whl (30 kB)
Requirement already satisfied: pycparser in /usr/lib/python3.9/site-packages (from cffi>=1.12->cryptography->ansible==2.9.24) (2.21)
Building wheels for collected packages: ansible, PyYAML
  Building wheel for ansible (setup.py): started
  Building wheel for ansible (setup.py): finished with status 'done'
  Created wheel for ansible: filename=ansible-2.9.24-py3-none-any.whl size=16205052 sha256=0105fa910a1fad576b8186e5e43f53222f17cb2d5077fe52f0db2497a8a20f43
  Stored in directory: /root/.cache/pip/wheels/ba/89/f3/df35238037ec8303702ddd8569ce11a807935f96ecb3ff6d52
  Building wheel for PyYAML (pyproject.toml): started
  Building wheel for PyYAML (pyproject.toml): finished with status 'done'
  Created wheel for PyYAML: filename=PyYAML-6.0-cp39-cp39-linux_x86_64.whl size=45332 sha256=f4700dcb5ff0141592227a3f9c318d03ba32af33ec45e6b480653de9ee75606f
  Stored in directory: /root/.cache/pip/wheels/b4/0f/9a/d6af48581dda678920fccfb734f5d9f827c6ed5b4074c7eda8
Successfully built ansible PyYAML
Installing collected packages: MarkupSafe, PyYAML, jinja2, ansible
Successfully installed MarkupSafe-2.0.1 PyYAML-6.0 ansible-2.9.24 jinja2-3.0.3
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Collecting mitogen
  Downloading mitogen-0.3.2-py2.py3-none-any.whl (288 kB)
Collecting ansible-lint
  Downloading ansible_lint-5.3.2-py3-none-any.whl (115 kB)
Collecting jmespath
  Downloading jmespath-0.10.0-py2.py3-none-any.whl (24 kB)
Requirement already satisfied: packaging in /usr/lib/python3.9/site-packages (from ansible-lint) (20.9)
Collecting wcmatch>=7.0
  Downloading wcmatch-8.3-py3-none-any.whl (42 kB)
Collecting tenacity
  Downloading tenacity-8.0.1-py3-none-any.whl (24 kB)
Requirement already satisfied: pyyaml in /usr/lib/python3.9/site-packages (from ansible-lint) (6.0)
Collecting enrich>=1.2.6
  Downloading enrich-1.2.7-py3-none-any.whl (8.7 kB)
Collecting rich>=9.5.1
  Downloading rich-11.0.0-py3-none-any.whl (215 kB)
Collecting ruamel.yaml<1,>=0.15.37
  Downloading ruamel.yaml-0.17.20-py3-none-any.whl (109 kB)
Collecting commonmark<0.10.0,>=0.9.0
  Downloading commonmark-0.9.1-py2.py3-none-any.whl (51 kB)
Collecting pygments<3.0.0,>=2.6.0
  Downloading Pygments-2.11.2-py3-none-any.whl (1.1 MB)
Requirement already satisfied: colorama<0.5.0,>=0.4.0 in /usr/lib/python3.9/site-packages (from rich>=9.5.1->ansible-lint) (0.4.4)
Collecting ruamel.yaml.clib>=0.2.6
  Downloading ruamel.yaml.clib-0.2.6.tar.gz (180 kB)
  Preparing metadata (setup.py): started
  Preparing metadata (setup.py): finished with status 'done'
Collecting bracex>=2.1.1
  Downloading bracex-2.2.1-py3-none-any.whl (12 kB)
Building wheels for collected packages: ruamel.yaml.clib
  Building wheel for ruamel.yaml.clib (setup.py): started
  Building wheel for ruamel.yaml.clib (setup.py): finished with status 'done'
  Created wheel for ruamel.yaml.clib: filename=ruamel.yaml.clib-0.2.6-cp39-cp39-linux_x86_64.whl size=746357 sha256=68562a65e781f0d8b9256c4170f4e966be549fc64df1f7a726fe61a31d401815
  Stored in directory: /root/.cache/pip/wheels/b1/c4/5d/d96e5c09189f4d6d2a9ffb0d7af04ee06d11a20f613f5f3496
Successfully built ruamel.yaml.clib
Installing collected packages: pygments, commonmark, ruamel.yaml.clib, rich, bracex, wcmatch, tenacity, ruamel.yaml, enrich, mitogen, jmespath, ansible-lint
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
Successfully installed ansible-lint-5.3.2 bracex-2.2.1 commonmark-0.9.1 enrich-1.2.7 jmespath-0.10.0 mitogen-0.3.2 pygments-2.11.2 rich-11.0.0 ruamel.yaml-0.17.20 ruamel.yaml.clib-0.2.6 tenacity-8.0.1 wcmatch-8.3
Collecting pywinrm
  Downloading pywinrm-0.4.2-py2.py3-none-any.whl (44 kB)
Requirement already satisfied: six in /usr/lib/python3.9/site-packages (from pywinrm) (1.15.0)
Collecting xmltodict
  Downloading xmltodict-0.12.0-py2.py3-none-any.whl (9.2 kB)
Requirement already satisfied: requests>=2.9.1 in /usr/lib/python3.9/site-packages (from pywinrm) (2.25.1)
Collecting requests-ntlm>=0.3.0
  Downloading requests_ntlm-1.1.0-py2.py3-none-any.whl (5.7 kB)
Requirement already satisfied: chardet<5,>=3.0.2 in /usr/lib/python3.9/site-packages (from requests>=2.9.1->pywinrm) (4.0.0)
Requirement already satisfied: idna<3.3,>=2.5 in /usr/lib/python3.9/site-packages (from requests>=2.9.1->pywinrm) (3.2)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /usr/lib/python3.9/site-packages (from requests>=2.9.1->pywinrm) (1.26.5)
Requirement already satisfied: certifi>=2017.4.17 in /usr/lib/python3.9/site-packages (from requests>=2.9.1->pywinrm) (2020.12.5)
Collecting ntlm-auth>=1.0.2
  Downloading ntlm_auth-1.5.0-py2.py3-none-any.whl (29 kB)
Requirement already satisfied: cryptography>=1.3 in /usr/lib/python3.9/site-packages (from requests-ntlm>=0.3.0->pywinrm) (36.0.1)
Requirement already satisfied: cffi>=1.12 in /usr/lib/python3.9/site-packages (from cryptography>=1.3->requests-ntlm>=0.3.0->pywinrm) (1.15.0)
Requirement already satisfied: pycparser in /usr/lib/python3.9/site-packages (from cffi>=1.12->cryptography>=1.3->requests-ntlm>=0.3.0->pywinrm) (2.21)
Installing collected packages: ntlm-auth, xmltodict, requests-ntlm, pywinrm
Successfully installed ntlm-auth-1.5.0 pywinrm-0.4.2 requests-ntlm-1.1.0 xmltodict-0.12.0
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
WARNING: Ignoring https://dl-cdn.alpinelinux.org/alpine/v3.14/main: No such file or directory
WARNING: Ignoring https://dl-cdn.alpinelinux.org/alpine/v3.14/community: No such file or directory
(1/37) Purging build-dependencies (20220127.065221)
(2/37) Purging python3-dev (3.9.5-r2)
(3/37) Purging libffi-dev (3.3-r2)
(4/37) Purging linux-headers (5.10.41-r0)
(5/37) Purging cargo (1.52.1-r1)
(6/37) Purging rust (1.52.1-r1)
(7/37) Purging rust-stdlib (1.52.1-r1)
(8/37) Purging openssl-dev (1.1.1l-r0)
(9/37) Purging libressl-dev (3.3.3-r0)
(10/37) Purging libressl3.3-libssl (3.3.3-r0)
(11/37) Purging libressl3.3-libtls (3.3.3-r0)
(12/37) Purging build-base (0.5-r2)
(13/37) Purging file (5.40-r1)
(14/37) Purging g++ (10.3.1_git20210424-r2)
(15/37) Purging gcc (10.3.1_git20210424-r2)
(16/37) Purging binutils (2.35.2-r2)
(17/37) Purging libatomic (10.3.1_git20210424-r2)
(18/37) Purging libgomp (10.3.1_git20210424-r2)
(19/37) Purging libgphobos (10.3.1_git20210424-r2)
(20/37) Purging make (4.3-r0)
(21/37) Purging libc-dev (0.7.2-r3)
(22/37) Purging musl-dev (1.2.2-r3)
(23/37) Purging fortify-headers (1.1-r1)
(24/37) Purging patch (2.7.6-r7)
(25/37) Purging pkgconf (1.7.4-r0)
(26/37) Purging mpc1 (1.2.1-r0)
(27/37) Purging mpfr4 (4.1.0-r0)
(28/37) Purging isl22 (0.22-r0)
(29/37) Purging gmp (6.2.1-r0)
(30/37) Purging llvm11-libs (11.1.0-r2)
(31/37) Purging libxml2 (2.9.12-r1)
(32/37) Purging libgit2 (1.1.0-r2)
(33/37) Purging http-parser (2.9.4-r0)
(34/37) Purging pcre (8.44-r0)
(35/37) Purging libssh2 (1.9.0-r1)
(36/37) Purging libressl3.3-libcrypto (3.3.3-r0)
(37/37) Purging libmagic (5.40-r1)
Executing busybox-1.33.1-r6.trigger
OK: 98 MiB in 69 packages
Removing intermediate container 07940c5f97b4
 ---> dea58b823135
Step 3/5 : RUN mkdir /ansible &&     mkdir -p /etc/ansible &&     echo 'localhost' > /etc/ansible/hosts
 ---> Running in f903ecbc1562
Removing intermediate container f903ecbc1562
 ---> 4336a5789527
Step 4/5 : WORKDIR /ansible
 ---> Running in ac6f3990cd71
Removing intermediate container ac6f3990cd71
 ---> 51fc9ee4057a
Step 5/5 : CMD [ "ansible-playbook", "--version" ]
 ---> Running in cfc9b1e46d78
Removing intermediate container cfc9b1e46d78
 ---> 375f0c81e1f6
Successfully built 375f0c81e1f6
Successfully tagged olegbukatchuk/ansible:2.9.24
```
</details>

Проверяем образ:
```bash
gnoy@manjarokde-ws01 ansible]$ docker image ls
REPOSITORY              TAG       IMAGE ID       CREATED         SIZE
olegbukatchuk/ansible   2.9.24    375f0c81e1f6   3 minutes ago   227MB
debian                  latest    04fbdaf87a6a   29 hours ago    124MB
alpine                  3.14      0a97eee8041e   2 months ago    5.6MB
centos                  latest    5d0da3dc9764   4 months ago    231MB
```
Пушим получившейся образ к себе в публичное репо:
```bash
[gnoy@manjarokde-ws01 ansible]$ docker login
Authenticating with existing credentials...
WARNING! Your password will be stored unencrypted in /home/gnoy/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[gnoy@manjarokde-ws01 ansible]$ docker tag olegbukatchuk/ansible:2.9.24 gnoy4eg/ansible:2.9.24
[gnoy@manjarokde-ws01 ansible]$ docker push gnoy4eg/ansible:2.9.24
The push refers to repository [docker.io/gnoy4eg/ansible]
2e2be001a53f: Pushed 
b45b5a2cd299: Pushed 
1a058d5342cc: Mounted from library/alpine 
2.9.24: digest: sha256:c1c703b2872ef0e87d62d2ab1204bc12ed3087fea89377a84af2a593e240d71b size: 947
```
Проверяем репозиторий в браузере:  
![repo-screenshot](https://github.com/gnoy4eg/netology.devops.pub/blob/main/img/repo_hubdockercom.png)