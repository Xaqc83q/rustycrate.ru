---
title: >
  Как настроить сборку и тестирование для Open Source проекта на Rust под Linux
  с помощью Travis
author: Михаил Панков
categories: обучение
---

{% img '2017-07-30-rust-travis/teaser.png' alt:'teaser' width:50% %}

Как зарегистрироваться на Travis, подключить туда свой проект на Rust и сделать
первую сборку.

<!--cut-->

# Содержание

* TOC
{:toc}

# Требования

* Исходники проекта запушены на GitHub

# Настройка простейшего проекта

В качестве примера здесь фигурирует репозиторий простейшей
библиотеки [hello](https://github.com/mkpankov/hello), состоящей из одной
функции и одного теста.

## Регистрируемся на Travis

Заходим на [Travis](https://travis-ci.org/). Нажимаем "Sign Up".

Если вы не вошли в свой аккаунт на GitHub, вам предложат войти:

{% img '2017-07-30-rust-travis/github-login.png' alt:'вход на GitHub' %}

После этого Travis запросит разрешение на доступ к данным на GitHub. Разрешите
его.

Если всё успешно, вы увидите панель управления Travis:

{% img '2017-07-30-rust-travis/dashboard.png' alt:'панель управления' %}

Если что-то пошло не так:

1. Отключите блокировку кук и скриптов в браузере.
2. Удалите куки.
3. Отзовите у Travis доступ к GitHub
   на [странице настроек](https://github.com/settings/applications) (кнопка
   Revoke).
4. Попробуйте ещё раз.

## Добавляем проект

Нажмите "+" над списком проектов в левой части панели управления:

{% img '2017-07-30-rust-travis/add-project.png' alt:'кнопка добавления проекта' %}

Появится окно добавления проектов. В нём найдите ваш проект и кликните по
переключателю для подключения проекта к Travis:

{% img '2017-07-30-rust-travis/enable-project.gif' alt:'кнопка включения проекта' width:50% %}

Если вы не видите ваш проект в списке, нажмите кнопку "Sync account" справа
вверху.

## Добавляем .travis.yml

Создаём
файл
[`.travis.yml`](https://github.com/mkpankov/hello/blob/2cde99cbb83721c0ae3308c8d519088a55c30445/.travis.yml) в
вашем репозитории. Там нам нужна всего одна строка:

```yaml
language: rust
```

Коммитим его:

```shell
$ git add .travis.tml
$ git commit -m "Добавляем Travis"
```

## Пушим на GitHub

Travis запускает сборки по пушу в репозиторий. Когда мы запушим наш коммит с
`.travis.yml`, начнётся первая сборка. Сделаем это:

```shell
$ git push
```

и идём в панель управления Travis. Находим там слева наш проект, кликаем по
нему.

Обычно нужно немного подождать, прежде чем начнётся сборка - в пределах
минуты.

Если вы видите в Travis свой коммит, но логов сборки внизу нет ("Hang tight, the
log cannot be shown until the build has started.") - подождите подольше. Это
значит, что сборка коммита поставлена в очередь. В пиковое время ждать
приходится порядка 5 минут. Это ограничение бесплатного тарифа Travis.

Когда сборка запустится, вы увидите логи внизу:

{% img '2017-07-30-rust-travis/building.png' alt:'сборка' %}

Travis собирает проект и запускает его тесты, аналогично тому, как это делается
на локальной машине.

Весь процесс занимает примерно минуту для простейшего проекта.

Вот как выглядит
страница [сборки](https://travis-ci.org/mkpankov/hello/builds/258901767) для
моей библиотеки hello.

Здесь если кликнуть по маленькому треугольнику слева от названия шага, скрытый в
нём вывод будет показан:

{% img '2017-07-30-rust-travis/expand.gif' alt:'раскрытие шага' %}

Чтобы получить полный лог в виде текстового файла, кликните по "Raw log" справа
вверху, над логом.

После окончания сборки вам на почту должно прийти письмо с результатом:

{% img '2017-07-30-rust-travis/email.png' alt:'email' %}

По умолчанию, такие письма приходят на каждую проваленную сборку, а также когда
сборка была сломанной и стала успешной.

## Настраиваем кэширование

Каждая сборка происходит в новом чистом Docker-контейнере. Поэтому компилятор и
крейты скачиваются каждый раз заново, а сборочная директория очищается.

Чтобы ускорить повторную сборку, укажем, что нужно настроить кэш Travis под
Cargo:

`.travis.yml`

```yaml
cache: cargo
```

Так мы добавляем в кэш `$HOME/.cargo` и `$TRAVIS_BUILD_DIR/target`.

Travis рекомендует не кэшировать вещи, которые долго скачиваются, но быстро
устанавливаются. Упаковка кэшируемых директорий в архив и их загрузка на
кэширующий сервер тоже занимает время, поэтому если просто загружать ресурсы из
интернета в контейнер без кэширования, это будет быстрее - не тратится время на
упаковку-распаковку архива кэша. По этой причине не стоит кэшировать директорию
установки компилятора (`$HOME/.rustup`).

## Добавляем индикатор статуса сборки

Чтобы видеть статус сборки на странице проекта, добавим индикатор в наш
репозиторий.

Заходим на страницу проекта в Travis, нажимаем на "build passing" вверху по
центру, выбираем там Markdown:

{% img '2017-07-30-rust-travis/badge.gif' alt:'код для индикатора' %}

Скопированный текст вставляем в `README.md` в нашем репозитории:

`README.md`

``` markdown
[![Build Status](https://travis-ci.org/mkpankov/hello.svg?branch=master)](https://travis-ci.org/mkpankov/hello)
```

Идём на [страницу проекта в GitHub](https://github.com/mkpankov/hello), видим
там индикатор:

{% img '2017-07-30-rust-travis/badge-in-repo.png' alt:'индикатор на GitHub' %}

## Готово!

Мы настроили сборку и тестирование проекта на Rust на Travis. Он будет
тестироваться на каждый пуш в репозиторий.

# Продвинутые возможности

## Выбор версии Rust

Так можно указать необходимую версию компилятора:

`.travis.yml`
``` yaml
language: rust
rust:
  - 1.10.0
```

## Тестирование с несколькими версиями

Можно собирать и тестировать проект с несколькими версиями компилятора:

`.travis.yml`
``` yaml
language: rust
rust:
  - stable
  - beta
  - nightly
matrix:
  allow_failures:
    - rust: nightly
```

Такая конфигурация последовательно соберёт и запустит тесты на stable, beta и
nightly. При этом ошибки во время сборки или тестирования на nightly не приведут
к провалу всей задачи на Travis: в индикаторе всё равно будет написано "passing".

## Как поменять шаги тестирования

По умолчанию Travis выполняет такие шаги для проектов на Rust:

``` shell
$ cargo build --verbose
$ cargo test --verbose
```

Вы можете перегрузить используемый скрипт. Например, если ваш проект
использует
[Cargo workspaces](https://doc.rust-lang.org/book/second-edition/ch14-03-cargo-workspaces.html),
при сборке нужно передавать опцию `--all`. Это можно сделать так:

`.travis.yml`
``` yaml
language: rust
script:
  - cargo build --verbose --all
  - cargo test --verbose --all
```

<hr/>

На этом всё. Успешных сборок!

Задавайте
вопросы
[на форуме](https://forum.rustycrate.ru/t/obsuzhdenie-stati-kak-nastroit-sborku-i-testirovanie-dlya-open-source-proekta-na-rust-pod-linux-s-pomoshhyu-travis/240).