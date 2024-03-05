# Справочная система Linux 

## Команда man

Основная встроенная справочная система Linux - это man-страницы. Чтобы получить информацию по команде (или программе, или файлу и т.д.) нужно ввести [`man`](https://manpages.ubuntu.com/manpages/jammy/ru/man1/man.1.html) и название интересующего нас объекта:

```bash
man man
man ls
man sshd_config
```

В результате откроется страница со справочной информацией по `man`, `ls`, `sshd_config` соответственно. Чтобы её закрыть нажмите `q`.

Сам по себе `man` занимается только поиском и отображением страниц, которые обычно хранятся на диске в каталоге `/usr/share/man`. Эти же страницы можно посмотреть в браузере на сайте https://manpages.ubuntu.com/ в формате html-страниц.

Несмотря на то, что Ubuntu уже довольно много лет man-страниц на русском языке всё ещё очень мало и как вариант можно воспользоваться: http://www.opennet.ru/man.shtml

Если во время установки вы выбрали (как вариант) английскую локализацию дистрибутива, то справочная информация будет отображаться на английском языке. Чтобы `man` показал справку на другом языке (например на руссом), можно запустить команду с ключом `-L`:

```bash
man -L ru man
```

Если страница для указанного языка существует, то будет показана она иначе, её вариант на английском.

### Задание

1. При помощи `man` изучите справочную информацию по команде `man`;
2. Найдите команду для которой доступен перевод на русский язык (любую кроме команды `man`) и запустите `man` для этой команды с явным указание языка;
3. Информацию для этой же команды найдите на сайте https://manpages.ubuntu.com/.

<br>

## Команды apropos и whatis

Может возникнуть ситуация, когда требуется выполнить какую-либо задачу, а есть ли программа для этого, и как она называется, неизвестно. В таких случаях можно попробовать поискать их с помощью утилит [`apropos`](https://manpages.ubuntu.com/manpages/jammy/en/man1/apropos.1.html) или [`whatis`](https://manpages.ubuntu.com/manpages/xenial/ru/man1/whatis.1.html).

Первая из них выполняет поиск по названиям и описаниям man-страниц и показывает список совпадений. Например, нам нужно изменить пароль пользователя. Тогда поиск:

```bash
apropos -a change pass
```

покажет:

```bash
chage (1)            - change user password expiry information
cryptsetup-luksChangeKey (8) - change an existing passphrase
passwd (1)           - change user password
sg_emc_trespass (8)  - change ownership of SCSI LUN from another Service-Proc...
```

Из вывода понятно, что нам подойдёт команда `passwd`.  
Здесь ключ `-a` использовался, чтобы отобразить только те результаты, которые содержать оба искомых слова. Иначе для совпадения будет достаточно любого из перечисленных слов.

Команда  `whatis` ищет совпадение только по названиям man-страниц и подходит в тех случаях, когда вы хотите вспомнить, что команда делает, или помните только часть команды и хотите посмотреть список похожих. Например, вы забыли что делает `ls`, тогда:

```bash
whatis ls
```

покажет:

```bash
ls (1)               - list directory contents
```

Или хотите посмотреть какие команды начинаются с pass:

```bash
whatis -w "pass*"
```

Вывод:

```bash
passphrase-encoding (7ssl) - How diverse parts of OpenSSL treat pass phrases ...
passwd (1)           - change user password
passwd (1ssl)        - OpenSSL application commands
passwd (5)           - the password file
```

Здесь добавлен ключ `-w`, чтобы текст воспринимался как шаблон, а не как точное название. Шаблон для поиска нужно заключить в кавычки.

Как видно в последнем выводе название `passwd` встречается несколько раз. Но если просто ввести `man passwd` откроется описание passwd (1) . Чтобы уточнить номер раздела, можно написать так: `man passwd.5`.

### Задание

4. Воспользуйтесь командой `apropos` и найдите команду для добавления пользователя в группу. В процессе поиска вводить название команды нельзя, только описание;
5. Воспользуйтесь командой `whatis` и найдите команды, название которых содержит `grp` в любом месте (в начале, середине, конце).

<br>

## GNU info

Другой источник информации о Linux и составляющих его программах — справочная подсистема [`info`](https://manpages.ubuntu.com/manpages/mantic/en/man1/info.1.html). Её можно представить как одну большую книгу, с оглавлением и ссылками для удобства перемещения. Чтобы воспользоваться ссылкой, нужно передвинуть курсор в любое её место и нажать Enter.

В остальном использование `info` похоже на `man`:

```bash
info ls
```


### Задание

6. Изучите справке по команде `info` при помощи команды `info`;
7. Изучите справочную информацию `info` для команды, которую вы выбрали в пункте 2.

<br>

## Ключи --help -h

Часто справочную информацию можно получить запустив нужную программу к ключом `--help` или `-h`. Вывод может быть таким же как и в `man` или содержать что-нибудь другое: 

```bash
ls --help
```

### Задание

8. Изучите справочную информацию доступную по ключу `--help` для команды, которую вы выбрали в пункте 2;
9. Сделайте вывод о том, насколько похожа информация из этих трёх источников.

<br>

## Альтернатива

Если некоторый объект системы не имеет документации ни в формате `man`, ни в формате `info`, то можно надеяться, что при нём есть сопроводительная документация в каталоге `/usr/share/doc/имя_объекта`. У этой документации нет единого формата или способа просмотра. Как правило это просто набор файлов, в котором нужно разбираться "руками".

<br>

## Источники

- [Основы Linux от основателя Gentoo. Часть 3 (1/4): Документация](https://habr.com/ru/articles/108764/);