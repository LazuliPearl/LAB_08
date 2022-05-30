Laboratory work VIII
Данная лабораторная работа посвещена изучению систем автоматизации развёртывания и управления приложениями на примере Docker

$ open https://docs.docker.com/get-started/
Tasks
 1. Создать публичный репозиторий с названием lab08 на сервисе GitHub
 2. Ознакомиться со ссылками учебного материала
 3. Выполнить инструкцию учебного материала
 4. Составить отчет и отправить ссылку личным сообщением в Slack
Report
Клонируем репозиторий с прошлой лр

$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab08
$ cd lab08
$ git submodule update --init
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab08
Создаем файл для Docker

$ cat > Dockerfile <<EOF
FROM ubuntu:18.04
EOF
Устанавливаем компиляторы и cmake

$ cat >> Dockerfile <<EOF

RUN apt update
RUN apt install -yy gcc g++ cmake
EOF
Копируем все полученные файлы в новую директорию print и делаем ее активной (аналог cd)

$ cat >> Dockerfile <<EOF

COPY . print/
WORKDIR print
EOF
Выполняем сборку нашего приложения

$ cat >> Dockerfile <<EOF

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_install
RUN cmake --build _build
RUN cmake --build _build --target install
EOF
Добавляем переменную окружения для работы программы demo/main.cpp

$ cat >> Dockerfile <<EOF

ENV LOG_PATH /home/logs/log.txt
EOF
Указываем каталог, изменения в котором будут сохранены

$ cat >> Dockerfile <<EOF

VOLUME /home/logs
EOF
Переходим в созданную на этапе сборки директорию

$ cat >> Dockerfile <<EOF

WORKDIR _install/bin
EOF
Запускаем исполняемый файл приложения

$ cat >> Dockerfile <<EOF

ENTRYPOINT ./demo
EOF
Дописываем эти строки в наш CMakeLists.txt, чтобы задать правила для сборки нового приложения demo/main.cpp

...
add_executable(demo ${CMAKE_CURRENT_SOURCE_DIR}/demo/main.cpp)
target_link_libraries(demo print) 
install(TARGETS demo RUNTIME DESTINATION bin)
...
Собираем образ

$ docker build -t logger .
Команда для просмотра доступных образов

$ docker images
Вызов нашей программы из образа (Флаг -i — это сокращение для --interactive. Благодаря этому флагу поток STDIN поддерживается в открытом состоянии даже если контейнер к STDIN не подключён. Флаг -t — это сокращение для --tty. Благодаря этому флагу выделяется псевдотерминал, который соединяет используемый терминал с потоками STDIN и STDOUT контейнера. Параметр -v задает каталог аналогично volume)

$ mkdir logs
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger
#Вводим текст и жмем CTRL-D
Команда для просмотра состояния образа

$ docker inspect logger
Проверим, что программа вывела наш текст в лог

$ cat logs/log.txt
Создадим .yml файл для создания образа

$ vim .travis.yml
/lang<CR>o
services:
- docker<ESC>
jVGdo
script:
- docker build -t logger .<ESC>
:wq
Пуш всех изменений

$ git add Dockerfile
$ git add .travis.yml
$ git commit -m"adding Dockerfile"
$ git push origin master
