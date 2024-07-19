# Домашнее задание к занятию "`Индексы`" - `Клименко Олег`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

---

### Задание 1

Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

Для таблицы

```
SELECT table_name, CONCAT(ROUND(INDEX_LENGTH /DATA_LENGTH*100), ' %') AS percent
FROM INFORMATION_SCHEMA.TABLES
WHERE table_name = "city";
```

![](https://cdn.discordapp.com/attachments/1258765922269925376/1263821221330423890/1.png?ex=669ba0bb&is=669a4f3b&hm=2703ad8fcb7606c5863c88eab80652e3085c1ea97fbdc3ed5daab22e3c3fee78&)

Для всех таблиц

```
SELECT CONCAT(ROUND(SUM(INDEX_LENGTH) / SUM(DATA_LENGTH) * 100), ' %') AS percent
FROM INFORMATION_SCHEMA.TABLES;
```

![](https://cdn.discordapp.com/attachments/1258765922269925376/1263821386900574208/3.png?ex=669ba0e3&is=669a4f63&hm=bcfb3965c5c17e22ce9bbebb3717c9cd84b3a278f78b86c64ad8b77dbfda1f0e&)

---

### Исправления

```
SELECT CONCAT(ROUND(SUM(INDEX_LENGTH) / (SUM(INDEX_LENGTH)+SUM(DATA_LENGTH)) * 100), ' %') AS percent
FROM INFORMATION_SCHEMA.TABLES;
```

![](https://cdn.discordapp.com/attachments/1258765922269925376/1263821547290759239/4.png?ex=669ba109&is=669a4f89&hm=3b6b5e00272d2546fc279453305e5789988612c2ea1f6a3b87e4edd17b347166&)

---

### Задание 2

Выполните explain analyze следующего запроса:

```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```

- перечислите узкие места;
- оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.

```
-> Limit: 400 row(s)  (cost=0..0 rows=0) (actual time=7312..7312 rows=391 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=7312..7312 rows=391 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=7312..7312 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id,f.title )   (actual time=3014..7033 rows=642000 loops=1)
                -> Sort: c.customer_id, f.title  (actual time=3014..3139 rows=642000 loops=1)
                    -> Stream results  (cost=21.2e+6 rows=16e+6) (actual time=0.626..2102 rows=642000 loops=1)
                        -> Nested loop inner join  (cost=21.2e+6 rows=16e+6) (actual time=0.619..1743 rows=642000 loops=1)
                            -> Nested loop inner join  (cost=19.6e+6 rows=16e+6) (actual time=0.614..1521 rows=642000 loops=1)
                                -> Nested loop inner join  (cost=18e+6 rows=16e+6) (actual time=0.607..1253 rows=642000 loops=1)
                                    -> Inner hash join (no condition)  (cost=1.58e+6 rows=15.8e+6) (actual time=0.589..68.4 rows=634000 loops=1)
                                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1.65 rows=15813) (actual time=0.0555..8.8 rows=634 loops=1)
                                            -> Table scan on p  (cost=1.65 rows=15813) (actual time=0.0404..5.82 rows=16044 loops=1)
                                        -> Hash
                                            -> Covering index scan on f using idx_title  (cost=111 rows=1000) (actual time=0.0579..0.392 rows=1000 loops=1)
                                    -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.938 rows=1.01) (actual time=0.00114..0.00165 rows=1.01 loops=634000)
                                -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=250e-6 rows=1) (actual time=191e-6..220e-6 rows=1 loops=642000)
                            -> Single-row covering index lookup on i using PRIMARY (inventory_id=r.inventory_id)  (cost=250e-6 rows=1) (actual time=149e-6..178e-6 rows=1 loops=642000)
```

В выводе видим что мы сортируем по 2-м большим таблицам и проходим по большому количеству строк. Так как дополнительная сортировка по названию фильмов нигде больше не используется, то уберем ее и таблицу film. Еще отсутствует необходимость в таблице inventory и проверке условия inventory_id

```
EXPLAIN ANALYZE select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id)
from payment p, rental r, customer c
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id
```

```
-> Limit: 400 row(s)  (cost=0..0 rows=0) (actual time=14.9..15.1 rows=391 loops=1)
    -> Table scan on <temporary>  (cost=2.5..2.5 rows=0) (actual time=14.9..15 rows=391 loops=1)
        -> Temporary table with deduplication  (cost=0..0 rows=0) (actual time=14.9..14.9 rows=391 loops=1)
            -> Window aggregate with buffering: sum(payment.amount) OVER (PARTITION BY c.customer_id )   (actual time=13.2..14.6 rows=642 loops=1)
                -> Sort: c.customer_id  (actual time=13.1..13.2 rows=642 loops=1)
                    -> Stream results  (cost=23632 rows=16003) (actual time=0.127..12.9 rows=642 loops=1)
                        -> Nested loop inner join  (cost=23632 rows=16003) (actual time=0.12..12.4 rows=642 loops=1)
                            -> Nested loop inner join  (cost=18031 rows=16003) (actual time=0.11..11.4 rows=642 loops=1)
                                -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1606 rows=15813) (actual time=0.0887..9.33 rows=634 loops=1)
                                    -> Table scan on p  (cost=1606 rows=15813) (actual time=0.0688..6.96 rows=16044 loops=1)
                                -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.938 rows=1.01) (actual time=0.0021..0.00286 rows=1.01 loops=634)
                            -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=0.00126..0.0013 rows=1 loops=642)
```

Основное назначение OVER PARTITION BY - это группировка по значению для sum, такая группировка оставляет все строки, GROUP BY сокращает количество строк в запросе с помощью их группировки, попробуем использовать GROUP BY

```
EXPLAIN ANALYZE select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id
GROUP BY c.customer_id;
```

```
-> Limit: 400 row(s)  (actual time=12.1..12.1 rows=391 loops=1)
    -> Sort with duplicate removal: `concat(c.last_name, ' ', c.first_name)`, `sum(p.amount)`  (actual time=12.1..12.1 rows=391 loops=1)
        -> Table scan on <temporary>  (actual time=11.8..11.8 rows=391 loops=1)
            -> Aggregate using temporary table  (actual time=11.8..11.8 rows=391 loops=1)
                -> Nested loop inner join  (cost=23632 rows=16003) (actual time=0.0901..10.7 rows=642 loops=1)
                    -> Nested loop inner join  (cost=18031 rows=16003) (actual time=0.0839..9.72 rows=642 loops=1)
                        -> Filter: (cast(p.payment_date as date) = '2005-07-30')  (cost=1606 rows=15813) (actual time=0.0682..7.8 rows=634 loops=1)
                            -> Table scan on p  (cost=1606 rows=15813) (actual time=0.0551..5.83 rows=16044 loops=1)
                        -> Covering index lookup on r using rental_date (rental_date=p.payment_date)  (cost=0.938 rows=1.01) (actual time=0.00198..0.00275 rows=1.01 loops=634)
                    -> Single-row index lookup on c using PRIMARY (customer_id=r.customer_id)  (cost=0.25 rows=1) (actual time=0.00123..0.00127 rows=1 loops=642)
```

Получаем запрос

```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, rental r, customer c
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id
GROUP BY c.customer_id;
```

![](https://cdn.discordapp.com/attachments/1258765922269925376/1263822306199736410/image.png?ex=669ba1be&is=669a503e&hm=369bf4c89db5d1592f1be051d63de6483996169280527b89213be697f3323a4c&)

---

### Исправления

```
CREATE INDEX inx_pay_date ON payment(payment_date);

EXPLAIN ANALYZE select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount)
from payment p, customer c
where payment_date >= '2005-07-30' and payment_date < '2005-07-31' and p.customer_id = c.customer_id 
GROUP BY p.customer_id;
```

```
-> Limit: 400 row(s)  (actual time=4.11..4.2 rows=391 loops=1)
    -> Sort with duplicate removal: `concat(c.last_name, ' ', c.first_name)`, `sum(p.amount)`  (actual time=4.11..4.16 rows=391 loops=1)
        -> Table scan on <temporary>  (actual time=3.72..3.8 rows=391 loops=1)
            -> Aggregate using temporary table  (actual time=3.72..3.72 rows=391 loops=1)
                -> Nested loop inner join  (cost=507 rows=634) (actual time=0.05..2.91 rows=634 loops=1)
                    -> Index range scan on p using inx_pay_date over ('2005-07-30 00:00:00' <= payment_date < '2005-07-31 00:00:00'), with index condition: ((p.payment_date >= TIMESTAMP'2005-07-30 00:00:00') and (p.payment_date < TIMESTAMP'2005-07-31 00:00:00'))  (cost=286 rows=634) (actual time=0.0385..1.7 rows=634 loops=1)
                    -> Single-row index lookup on c using PRIMARY (customer_id=p.customer_id)  (cost=0.25 rows=1) (actual time=0.00163..0.00167 rows=1 loops=634)
```

![](https://cdn.discordapp.com/attachments/1258765922269925376/1263822582659153972/image.png?ex=669ba200&is=669a5080&hm=20320c7886249be562d995b179fdac57f874a284b92b0f30ac478f9009eadc2f&)

---
---
