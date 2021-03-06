# Сложные выборки: JOIN

Оператор JOIN позволяет соединить две или больше таблиц.

# Датасет для экспериментов

Переходим в директорию postgres_interactions и запускаем построение докер-контейнера

<pre>
$ sudo docker-compose --project-name postgres-client -f docker-compose.yml run --rm postgres-client
</pre>


Проверим, что контейнер знает о наших данных, полученных на Kaggle и о скрипте load_data.sh

<pre>
/ # ls /data
credits.csv          keywords.csv         links.csv            links_small.csv      movies_metadata.csv  ratings.csv          ratings_small.csv    test.csv             test.json
/ # ls /home
load_data.sh
</pre>

Создаём таблицу в Postgres и заливаем туда данные из csv

<pre>
# sh /home/load_data.sh
Загружаем links.csv...
CREATE TABLE
COPY 45843
Загружаем ratings.csv...
CREATE TABLE
COPY 26024289
</pre>

<pre>
psql --host $APP_POSTGRES_HOST -U postgres
psql (10.4)
Type "help" for help.

postgres=# select * from public.links limit 1;
 movieid | imdbid  | tmdbid 
---------+---------+--------
       1 | 0114709 | 862
(1 row)

postgres=# select * from public.ratings limit 1;
 userid | movieid | rating | timestamp  
--------+---------+--------+------------
      1 |     110 |      1 | 1425941529
(1 row)

</pre>

# JOIN: примеры

Синтаксис JOIN: указать таблицу которую присоединяем и поле для соединения:

<pre>
postgres=# select *
postgres-# from public.links
postgres-# JOIN public.ratings ON links.movieid=ratings.movieid
postgres-# LIMIT 5;
 movieid | imdbid  | tmdbid | userid | movieid | rating | timestamp  
---------+---------+--------+--------+---------+--------+------------
     110 | 0112573 | 197    |      1 |     110 |      1 | 1425941529
     147 | 0112461 | 10474  |      1 |     147 |    4.5 | 1425942435
     858 | 0068646 | 238    |      1 |     858 |      5 | 1425941523
    1221 | 0071562 | 240    |      1 |    1221 |      5 | 1425941546
    1246 | 0097165 | 207    |      1 |    1246 |      5 | 1425941556
(5 rows)

</pre>

Видно, что в результирующем запросе столбцы из обеих таблиц

INNER JOIN выпиливает строки, для которых не нашлось ключа. LEFT JOIN (как и RIGHT JOIN) такие строки оставляет:

<pre>
postgres=# SELECT * FROM public.links LEFT JOIN public.ratings ON links.movieid=ratings.movieid WHERE ratings.movieid IS NULL LIMIT 5;
 movieid | imdbid  | tmdbid | userid | movieid | rating | timestamp 
---------+---------+--------+--------+---------+--------+-----------
  110399 | 0028646 | 60438  |        |         |        |          
   99899 | 0107519 | 128644 |        |         |        |          
  117103 | 0069961 | 184061 |        |         |        |          
  150950 | 0031406 | 133255 |        |         |        |          
  124791 | 0028367 | 149955 |        |         |        |          
(5 rows)
</pre>

OUTER JOIN выведет все строки, когда ключ есть хотя бы в одной таблице

Соединение двух и более таблиц происходит аналогично.
