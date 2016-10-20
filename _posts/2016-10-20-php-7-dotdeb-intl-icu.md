---
layout: post
title:  "Сборка intl под PHP 7 (Dotdeb) с актуальным ICU"
date:   2016-10-20 12:21:00 +0300
author: NeoFusion
---
Изначально пакет `php7.0-intl` собран с ICU 52.1.
Для использования c ICU 57.1 нам необходимо пересобрать его из исходников, чем мы сейчас и займемся.

<!--more-->

Текущая конфигурация:

  * Debian 8.6 (jessie);
  * PHP 7 из репозитория [Dotdeb](https://www.dotdeb.org/).

Для сборки нам необходимы следующие установленные пакеты:

    build-essential checkinstall git php7.0-dev

Сначала скачиваем и устанавливаем актуальный [ICU](http://site.icu-project.org/):

    wget http://download.icu-project.org/files/icu4c/57.1/icu4c-57_1-src.tgz
    tar xf icu4c-57_1-src.tgz
    cd icu/source/
    ./configure
    make install

Обязательно учитываем установленную версию из репозитория и указываем ее при checkout.
В нашем случае это `7.0.12`.

Выкачиваем исходники PHP 7 от Dotdeb и собираем `php7.0-intl` как deb-пакет:

    cd ~
    git clone https://github.com/gplessis/dotdeb-php.git
    cd dotdeb-php/
    git checkout upstream/7.0.12
    cd ext/intl/
    phpize7.0
    ./configure --prefix=/usr --with-icu-dir=/usr/local
    checkinstall make install

Отвечаем на вопросы `checkinstall`:

    n
    intl from source
    3
    7.0.12-by-root
    12
    php7.0-intl

В итоге получаются такие параметры:

    0 -  Maintainer: [ root@debian ]
    1 -  Summary: [ intl from source ]
    2 -  Name:    [ intl ]
    3 -  Version: [ 7.0.12-by-root ]
    4 -  Release: [ 1 ]
    5 -  License: [ GPL ]
    6 -  Group:   [ checkinstall ]
    7 -  Architecture: [ amd64 ]
    8 -  Source location: [ intl ]
    9 -  Alternate source location: [  ]
    10 - Requires: [  ]
    11 - Provides: [ intl ]
    12 - Conflicts: [ php7.0-intl ]
    13 - Replaces: [  ]

Созданный пакет будет установлен автоматически, если не возникло конфликтов.
Иначе необходимо будет удалить конфликтующие пакеты и выполнить установку командой:

    dpkg -i intl_7.0.12-by-root-1_amd64.deb

В папке `/etc/php/7.0/mods-available` создадим файл `intl.ini` со следующим содержимым:

    ; configuration for php intl module
    ; priority=20
    extension=intl.so

Активируем новый модуль:

    phpenmod intl
    systemctl restart apache2.service

В выводе `phpinfo()` теперь отображается новая версия:

    ICU version 57.1
