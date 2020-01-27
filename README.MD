# Визуализация логов, наука видеть иначе

# bash

### bash
HOW-TO: 10 простых и полезных трюков для командной строки UNIX и Linux
https://xakep.ru/2016/09/26/faq-10-bash-tricks/

Как поправить опечатку в команде


https://xakep.ru/2016/09/08/19-shell-scripts/
Конструкция для получения имени текущего каталога

### zsh

Прокачай терминал! Полезные трюки, которые сделают тебя гуру консоли

https://xakep.ru/2017/05/18/cli-console-tips/


Покоряем терминал. Как магия консоли позволяет сделать data engineering проще. Часть 2
https://xakep.ru/2016/06/07/console-magic-2/


Покоряем терминал. Как магия консоли позволяет сделать data engineering проще
https://xakep.ru/2016/05/17/console-magic/

### Mintty
Апгрейды для cmd.exe и альтернативы
https://xakep.ru/2012/11/22/upgrade-cmd/




# Консольные утилиты

### grep
Конкурс хаков: делаем рекурсивный поиск по файлам в каталогах при помощи grep и регулярок
https://xakep.ru/2017/09/19/hack-contest-grep-search/

Поиск файлов по маске, содержащих указанную подстроку
Прокачай терминал! Полезные трюки, которые сделают тебя гуру консоли

https://xakep.ru/2017/05/18/cli-console-tips/

### sed
http://www.grymoire.com/Unix/Sed.html

### Другие утилиты

https://xakep.ru/2018/06/25/www-structured-text-tools/
https://github.com/dbohdan/structured-text-tools

# AWK

Регулярные выражения в десятки раз медленее, чем деление по подстрокам.

Сервис для быстрого составления регулярных выражений

* http://txt2re.com
* https://xakep.ru/2017/11/03/www-txt2re/

# Логи

### В формате json

Для обработки логов подойдет команда jq
WWW: jq — процессор JSON, который позволяет одной строкой фильтровать и трансформировать данные
https://xakep.ru/2017/09/07/www-jq/


# Где и как найти примеры кода

https://xakep.ru/2015/11/26/searchcode/

https://searchcode.com

Подборки команд

https://github.com/chubin/cheat.sh

Рейтинг команд

https://www.commandlinefu.com/commands/browse/sort-by-votes/25
(ссылка из статьи https://xakep.ru/2016/09/26/faq-10-bash-tricks/)

# Книги и статьи

### Data Science at the Command Line

https://www.datascienceatthecommandline.com/


# Альтернативы

Log Stash - Google внутри компании, поиск по документам любого типа

https://xakep.ru/2017/04/06/elk-logs/

Он потребляет огромное количество ресурсов.
Не использую его, использую InfluxDB

============================

Перейдем к практике, используя Play Framework
[!img/play.framework.png]


============================

* https://github.com/playframework/play-scala-rest-api-example/tree/2.6.x
* https://github.com/polarnik/play-scala-rest-api-example/tree/2.6.x

**HTTP REST сервис**

Нагрузочный тест на Gatling

Копирование репозитория
```
git clone https://github.com/polarnik/play-scala-rest-api-example.git
sbt run
sbt gatling:test
```
=============================

Отправка метрик в InfluxDB
```
curl -i -XPOST "http://localhost:8086/write?db=telegraf" --data-binary @metrics.out
```
==================================

```
awk '{a[$3 " " $5]++}END{for(i in a){ print a[i] " " i} }' application.log | sort -rn 
    1860193 [TRACE] v1.post.PostActionBuilder
    1858454 [TRACE] v1.post.PostController
    1773001 [TRACE] v1.post.PostRepositoryImpl
    6912 [WARN] io.gatling.http.ahc.ResponseProcessor
    5  
    1 [INFO] play.api.Play
    1 [INFO] play.api.http.EnabledFilters
```

Или так
```
awk '{ print $3 " " $5 }' application.log | sort | uniq -c | sort -nr
1860193 [TRACE] v1.post.PostActionBuilder
1858454 [TRACE] v1.post.PostController
1773001 [TRACE] v1.post.PostRepositoryImpl
   6912 [WARN] io.gatling.http.ahc.ResponseProcessor
      5  
      1 [INFO] play.api.Play
      1 [INFO] play.api.http.EnabledFilters
```

Вывод сообщений об ошибке

```
awk -F 'failed: ' '{print $2}' application.log | sort | uniq -c | sort -nr
5491655 
   6459 j.n.ConnectException: connection timed out: localhost/127.0.0.1:9000
    445 j.u.c.TimeoutException: Request timeout to localhost/127.0.0.1:9000 after 60000 ms
      8 j.i.IOException: Соединение разорвано другой стороной
```

Для исключения пустой строки, добавим проверку на пустоту
```
awk -F 'failed: ' '{if ($2) print $2}' application.log | sort | uniq -c | sort -nr
```

Или можно добавить фильтрацию, выводить только те строки, что удовлетворяют регулярному выражению
```
awk -F 'failed: ' '/WARN/ {print $2}' application.log | sort | uniq -c | sort -nr 
   6459 j.n.ConnectException: connection timed out: localhost/127.0.0.1:9000
    445 j.u.c.TimeoutException: Request timeout to localhost/127.0.0.1:9000 after 60000 ms
      8 j.i.IOException: Соединение разорвано другой стороной
```

Или двум регулярным выражениям:
```
awk -F 'failed: ' '/WARN/ && /time/ {print $2}' application.log | sort | uniq -c | sort -nr 
   6459 j.n.ConnectException: connection timed out: localhost/127.0.0.1:9000
    445 j.u.c.TimeoutException: Request timeout to localhost/127.0.0.1:9000 after 60000 ms
```

Или можно составить сложный ключ группировки
```
~/Project/playframework/logs$ awk '{print $3 " " $5 " (" $9 " " $10")"}' application.log | sort | uniq -c | sort -nr

1860193 [TRACE] v1.post.PostActionBuilder (invokeBlock: )
1858454 [TRACE] v1.post.PostController (index: )
1773001 [TRACE] v1.post.PostRepositoryImpl (list: )
   6912 [WARN] io.gatling.http.ahc.ResponseProcessor (Request 'Index')
      5   ( )
      1 [INFO] play.api.Play (Application started)
      1 [INFO] play.api.http.EnabledFilters (Enabled Filters)
```

А что это за строки, которые не содержат тип сообщения? Давайте выведем их
```
~/Project/playframework/logs$ awk '{if ($3) {print $3 " " $5 " (" $9 " " $10")"} else {print $0}}' application.log | sort | uniq -c | sort -nr

1860193 [TRACE] v1.post.PostActionBuilder (invokeBlock: )
1858454 [TRACE] v1.post.PostController (index: )
1773001 [TRACE] v1.post.PostRepositoryImpl (list: )
   6912 [WARN] io.gatling.http.ahc.ResponseProcessor (Request 'Index')
      2 
      1     play.filters.hosts.AllowedHostsFilter
      1     play.filters.headers.SecurityHeadersFilter
      1     play.filters.csrf.CSRFFilter
      1 [INFO] play.api.Play (Application started)
      1 [INFO] play.api.http.EnabledFilters (Enabled Filters)
```
А давайте найдем, где эти строки спрятались
```
~/Project/playframework/logs$ grep -A 5 -B 5 play.filters application.log 

2018-09-10 01:48:52,505 [INFO] from play.api.http.EnabledFilters in play-dev-mode-akka.actor.default-dispatcher-8 - Enabled Filters (see <https://www.playframework.com/documentation/latest/Filters>):

    play.filters.csrf.CSRFFilter
    play.filters.headers.SecurityHeadersFilter
    play.filters.hosts.AllowedHostsFilter

2018-09-10 01:48:52,509 [INFO] from play.api.Play in play-dev-mode-akka.actor.default-dispatcher-8 - Application started (Dev)
2018-09-10 01:48:52,684 [TRACE] from v1.post.PostActionBuilder in application-akka.actor.default-dispatcher-2 - invokeBlock: 
2018-09-10 01:48:52,684 [TRACE] from v1.post.PostActionBuilder in application-akka.actor.default-dispatcher-5 - invokeBlock: 
2018-09-10 01:48:52,684 [TRACE] from v1.post.PostActionBuilder in application-akka.actor.default-dispatcher-3 - invokeBlock: 
```


