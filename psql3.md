# JOINS, отношения, практика запросов

## Отношения между таблицами

Как мы уже обсуждали, PostgreSQL относится к реляционным базам данных. Реляционные БД характеризуются наличием таблиц и отношений.

Существует 3 типа отношений:

- "**Один к одному**" - когда одна запись в первой таблице соответствует одной записи во второй таблице. Встречается такая связь довольно редко. Такая связь либо избыточна, т.е. может иметь смысл просто объединить данные в одну таблицу, либо это результат модернизации архитектуры, и такое решение кем-то принято обосновано.

![](https://zametkinapolyah.ru/wp-content/uploads/2016/05/%D0%BE%D0%B4%D0%B8%D0%BD-%D0%BA%D0%BE-%D0%BE%D0%B4%D0%BD%D0%BE%D0%BC%D1%83.png)

- "**Один ко многим**" - когда одной записи в первой таблице соответствует несколько записей в другой таблице. К примеру, у клиента магазина может быть несколько номеров. Один клиент - одна запись в таблице клиента, его номера телефонов - записи в таблице телефонов. Один клиент относится ко многим номерам телефонов, но обратная связь - номер телефона к клиенту - многие к одному.

![](https://zametkinapolyah.ru/wp-content/uploads/2016/05/%D0%9E%D0%B4%D0%B8%D0%BD-%D0%BA%D0%BE-%D0%BC%D0%BD%D0%BE%D0%B3%D0%B8%D0%BC.png)

- "**Многое ко многим**" - когда одной записи в первой таблице соответствует несколько записей во второй таблице, но одной записи второй таблицы может соответствовать несколько записей первой таблицы. Классический пример - книги и авторы, ведь у автора может быть много книг, и у каждой книги может быть несколько авторов. 
Реализуется за счет создания таблицы связей, куда копируются ключи записей обеих таблиц.

![](https://zametkinapolyah.ru/wp-content/uploads/2016/05/%D0%9C%D0%BD%D0%BE%D0%B3%D0%B8%D0%B5-%D0%BA%D0%BE-%D0%BC%D0%BD%D0%BE%D0%B3%D0%B8%D0%BC.png)

## Объединения (JOINS)


![JOINS](http://www.postgresqltutorial.com/wp-content/uploads/2018/12/PostgreSQL-Joins.png)
Запросы к одной таблице довольно редки. Чаще всего запросы к базам данных пишуться с целью получить информацию из нескольких таблиц, информация из которых объединяется по определенным условиям.

Создадим таблицы авторов, книг, жанров и таблицу связей для авторов и книг (многие ко многим):

```sql

CREATE TABLE authors (
	id SERIAL PRIMARY KEY,
	name VARCHAR(200) NOT NULL DEFAULT 'lore',
	year DATE NOT NULL DEFAULT '1970-01-01'
);

CREATE TABLE

CREATE TABLE books (
	id SERIAL PRIMARY KEY,
	title VARCHAR(200) NOT NULL DEFAULT 'noname',
	genre_id INT NOT NULL DEFAULT 0
);

CREATE TABLE

CREATE TABLE genres (
	id SERIAL PRIMARY KEY,
	genre VARCHAR(100) NOT NULL DEFAULT 'unknown'
);

CREATE TABLE

CREATE TABLE authors_books (
	author_id INT NOT NULL DEFAULT 0,
	book_id INT NOT NULL DEFAULT 0
);

CREATE TABLE
```

Далее следует заполнить таблицу данными:

```sql
INSERT INTO genres (genre) VALUES
	('SF'),
	('novel'),
	('story'),
	('horror');

INSERT INTO books(title, genre_id) VALUES
	('Мастер и Маргарита', 2),
	('Фауст', 0),
	('Белый клык', 3),
	('Дюна', 1),
	('Война и мир', 2);

INSERT INTO authors (name) VALUES
	('Френк Герберт'),
	('Михаил Булгаков'), 
	('Ждек Лондон'), 
	('Иоган Гёте'), 
	('Роберт Хайнлайн');

INSERT INTO authors_books (author_id, book_id) VALUES 
	(1, 4),
	(2, 1),
	(3, 3),
	(4, 2);

```

Данные готовы, теперь можно изучать объединения таблиц. Начнем мы с объединения двух таблиц в одном запросе:

```sql
SELECT title, genre
FROM books
INNER JOIN genres ON (genres.id = books.genre_id);

       title        | genre
--------------------+-------
 Мастер и Маргарита | novel
 Белый клык         | story
 Дюна               | SF
 Война и мир        | novel
(4 rows)

```

**`INNER JOIN`** выводит записи из левой таблицы (первой из двух таблиц, которые он объединяет), для которых найдется соответствующая запись в правой (второй и последней в списке) таблице. Если соответствия в правой таблице нет, такая запись не выводится. В данном случае можно видеть, что запись о книге "Фауст" не выводится. Это происходит из-за того, что `genre_id` у этой записи равен нулю, а такого жанра в таблице жанров нет.
Так же в результат не попадает жанр horror, так как нет ни единой книги с таким жанром.

```sql
SELECT title, genre
FROM books
LEFT JOIN genres ON (genres.id = books.genre_id);

       title        | genre
--------------------+-------
 Мастер и Маргарита | novel
 Фауст              |
 Белый клык         | story
 Дюна               | SF
 Война и мир        | novel

(5 rows)

```
**`LEFT JOIN`** выводит **ВСЕ** записи из левой таблицы. Для тех записей, которым находится соответствие в правой, он, аналогично **`INNER JOIN`**-у, выведет соответствующие данные из второй таблицы. Для тех же, которым соответствия не нашлось, он ничего не выведет в столбцах правой таблицы, оставив их пустыми.

Чтобы определить, каким книгам не найдены в соответствие жанры, можно выполнить запрос с подзапросом:

```sql
SELECT title
FROM books
WHERE NOT EXISTS (
	SELECT * 
	FROM genres 
	WHERE books.genre_id = genres.id
);

 title
-------
 Фауст
(1 row)
```

Этот путь считается более правильным, чем классический:

```sql
SELECT title, genre 
FROM books 
LEFT JOIN genres ON (genres.id = books.genre_id)
WHERE genre IS NULL;


 title
-------
 Фауст
(1 row)

```

**`RIGHT JOIN`** поступает аналогичным образом с правой таблицей: выводит все записи из нее, добавляя записи из левой. Где соответствия не находится, оставляет в столбцах левой таблицы пустоту.

```sql
SELECT title, genre
FROM books
RIGHT JOIN genres ON (genres.id = books.genre_id);

       title        | genre
--------------------+--------
 Мастер и Маргарита | novel
 Белый клык         | story
 Дюна               | SF
 Война и мир        | novel
                    | horror
(5 rows)```

Если у вас возникнет необходимость увидеть все книги и все жанры вне зависимости от наличия соответствующих записей в другой таблице, но где связи есть - связать таблицы, можно объединить результаты двух запросов:


```sql
SELECT title, genre 
FROM books 
LEFT JOIN genres ON (genres.id = books.genre_id) 
UNION 
SELECT title, genre 
FROM books RIGHT JOIN genres ON (genres.id = books.genre_id);


       title        | genre
--------------------+--------
 Фауст              |
                    | horror
 Война и мир        | novel
 Белый клык         | story
 Дюна               | SF
 Мастер и Маргарита | novel
(6 rows)
```

Аналогичного результата можно добиться и специальным видом объединений - **`FULL JOIN`**:

```sql
SELECT title, genre
FROM books
FULL JOIN genres ON (genres.id = books.genre_id);


       title        | genre
--------------------+--------
 Мастер и Маргарита | novel
 Фауст              |
 Белый клык         | story
 Дюна               | SF
 Война и мир        | novel
                    | horror
(6 rows)
```

Если поля для связи называются одинаково, например, поле id в таблице genres называется genre_id, запрос можно немного упростить. Для демонстрации переименуем поле и выполним этот запрос:

```sql
ALTER TABLE genres RENAME COLUMN id TO genre_id;

ALTER TABLE

SELECT title, genre
FROM books 
RIGHT JOIN genres USING(genre_id);


       title        | genre
--------------------+--------
 Мастер и Маргарита | novel
 Белый клык         | story
 Дюна               | SF
 Война и мир        | novel
                    | horror
(5 rows)


```

Результат будет тот же, что и у обычного **`RIGHT JOIN`** запроса, но сам запрос короче.

## Практика

- Объединить авторов с книгами посредством таблицы связей
- Построить фамильное древо по мужской линии

## Полезные ссылки

[Домашка](hw03.md)