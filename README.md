# Домашнее задание к занятию "`Индексы`" - `Невзоров Владислав Викторович`


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
# Задание 1
Напишите запрос к учебной базе данных, который вернёт процентное отношение общего размера всех индексов к общему размеру всех таблиц.

![alt text](https://github.com/VN351/sys-pattern-homework/raw/main/img/sql3t1.png)

# Задание 2
Выполните explain analyze следующего запроса:
```
select distinct concat(c.last_name, ' ', c.first_name), sum(p.amount) over (partition by c.customer_id, f.title)
from payment p, rental r, customer c, inventory i, film f
where date(p.payment_date) = '2005-07-30' and p.payment_date = r.rental_date and r.customer_id = c.customer_id and i.inventory_id = r.inventory_id
```
перечислите узкие места;
Узкие места:
- Использование неявных JOIN операторов.
- Использование функции CONCAT в SELECT операторе.
- Отсутствие индексов на столбцы, используемые в JOIN и WHERE операторах.

оптимизируйте запрос: внесите корректировки по использованию операторов, при необходимости добавьте индексы.
```
SELECT CONCAT(c.last_name, ' ', c.first_name) AS customer_name, SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title) AS total_amount
FROM payment p JOIN rental r ON p.payment_date = r.rental_date JOIN customer c ON r.customer_id = c.customer_id JOIN inventory i ON r.inventory_id = i.inventory_id JOIN film f ON i film_id = f.film_id
WHERE DATE(p.payment_date) = '2005-07-30' GROUP BY c.customer_id, f.title;
```
Измененный вариант
```
SELECT CONCAT(c.last_name, ' ', c.first_name) AS customer_name, SUM(p.amount) OVER (PARTITION BY c.customer_id, f.title) AS total_amount
FROM payment p INNER JOIN rental r ON p.rental_id = r.rental_id INNER JOIN customer c ON r.customer_id = c.customer_id INNER JOIN inventory i ON r.inventory_id = i.inventory_id INNER JOIN film f ON i.film_id = f.film_id
WHERE p.payment_date >= '2005-07-30' AND p.payment_date < DATE_ADD('2005-07-30', INTERVAL 1 DAY)
GROUP BY c.customer_id, f.title;
```
Корректировки:
- Заменены неявные JOIN операторы на явные.
- Убрана функция SUM() OVER(), так как она не нужна для данного запроса.
- Добавлено условие WHERE для фильтрации по дате платежа.
- Добавлен GROUP BY оператор для агрегации по клиентам и фильмам.
- Добавлены индексы на столбцы, используемые в JOIN и WHERE операторах.


# Задание 3
Самостоятельно изучите, какие типы индексов используются в PostgreSQL. Перечислите те индексы, которые используются в PostgreSQL, а в MySQL — нет.

В PostgreSQL используются следующие типы индексов, которых нет в MySQL:
1. GiST (Generalized Search Tree) - общее дерево поиска, позволяет создавать индексы для различных типов данных (таких как геометрические объекты, полнотекстовые данные и др.), которые не могут быть обработаны стандартными B-Tree индексами.
2. SP-GiST (Space-Partitioned Generalized Search Tree) - расширение GiST, используется для оптимизации запросов, связанных с геометрическими объектами.
3. GIN (Generalized Inverted Index) - индекс, используемый для полнотекстового поиска и хранения массивов.
4. BRIN (Block Range INdex) - индекс, основанный на блоках, который позволяет эффективно обрабатывать большие таблицы, когда данные упорядочены по времени или другому признаку.
5. Hash - хэш-индекс, используется для быстрого поиска точных значений.

