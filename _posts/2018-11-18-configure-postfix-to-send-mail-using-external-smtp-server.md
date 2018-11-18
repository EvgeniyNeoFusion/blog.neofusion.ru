---
layout: post
title:  "Настройка Postfix для отправки почты через внешний SMTP"
date:   2018-11-18 21:17:00 +0300
author: NeoFusion
---
В этом руководстве я опишу процесс настройки [Postfix](http://www.postfix.org/) для отправки (и переадресации локальной) почты через внешний SMTP сервер (на примере Gmail) с различными видами ограничений.

<!--more-->

В процессе написания руководства использовались:

  * Debian 7.6 (wheezy)
  * Postfix 2.9.6

В дальнейшем проверялось на версиях:

  * Debian 9.6 (stretch)
  * Postfix 3.1.8

#### Установка и настройка

Устанавливаем Postfix:

    aptitude install postfix

Тип настройки: "Только локальное использование".

В `/etc/postfix` создаем файл `sasl_passwd`, содержащий учетные данные для подключения к smtp:

    smtp.gmail.com noreply@example.com:password

Создаем базу:

    postmap sasl_passwd

В файле `main.cf` (или с помощью `postconf`) указываем следующие параметры:

    postconf -e "myhostname = localhost"
    postconf -e "inet_protocols = ipv4"
    postconf -e "default_transport = smtp"
    postconf -e "relay_transport = error"
    postconf -e "relayhost = smtp.gmail.com:587"
    postconf -e "smtp_sasl_auth_enable = yes"
    postconf -e "smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd"
    postconf -e "smtp_sasl_security_options = noanonymous"
    postconf -e "smtp_use_tls = yes"

Примерный вид получившегося файла `main.cf` (можно проверить также командой `postconf -n`):

    alias_database = hash:/etc/aliases
    alias_maps = hash:/etc/aliases
    append_dot_mydomain = no
    biff = no
    config_directory = /etc/postfix
    default_transport = smtp
    inet_interfaces = loopback-only
    inet_protocols = ipv4
    mailbox_command = procmail -a "$EXTENSION"
    mailbox_size_limit = 0
    mydestination = localhost, gitlab, localhost.localdomain
    myhostname = localhost
    mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
    readme_directory = no
    recipient_delimiter = +
    relay_transport = error
    relayhost = smtp.gmail.com:587
    smtp_sasl_auth_enable = yes
    smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
    smtp_sasl_security_options = noanonymous
    smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
    smtp_use_tls = yes
    smtpd_banner = $myhostname ESMTP $mail_name
    smtpd_tls_cert_file = /etc/ssl/certs/ssl-cert-snakeoil.pem
    smtpd_tls_key_file = /etc/ssl/private/ssl-cert-snakeoil.key
    smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
    smtpd_use_tls = yes

#### Список перенаправлений

В `/etc/postfix` создаем файл `virtual`, содержащий список перенаправлений с виртуальных адресов:

    user@localhost user@example.com

Создаем базу:

    postmap virtual

Добавляем параметр:

    postconf -e "virtual_alias_maps = hash:/etc/postfix/virtual"

#### Фильтрация rcpt to

Если необходима фильтрация rcpt to, то создаем файл `allowed_rcpt_domains`:

    localhost OK
    example.com OK

Создаем базу:

    postmap allowed_rcpt_domains

Добавляем параметр:

    postconf -e "smtpd_recipient_restrictions = check_recipient_access hash:/etc/postfix/allowed_rcpt_domains, reject"

#### Фильтрация с использованием transport_maps

Создаем файл `transport`:

    localhost local
    example.com smtp:smtp.gmail.com:587
    * error:Destination is not allowed

Создаем базу:

    postmap transport

Добавляем параметр:

    postconf -e "transport_maps = hash:/etc/postfix/transport"

#### Запуск и проверка

Перезапускаем Postfix:

    systemctl restart postfix.service

Проверяем отправку почты:

    echo "Test" | mail user@localhost

#### Разное

Различные примеры команд:

```shell
postconf         # Выводит все текущие параметры Postfix
postconf -n      # Выводит параметры содержащиеся в main.cf

postqueue -p     # Просмотр очереди сообщений
postsuper -d ALL # Удаление всех писем из очереди
```
