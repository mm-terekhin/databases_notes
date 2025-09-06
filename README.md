# Databases заметки

- [Databases заметки](#databases-заметки)
  - [Databases core](#databases-core)
    - [Основы работы с БД](#основы-работы-с-бд)
      - [1. 🔹 Создание базы данных](#1--создание-базы-данных)
      - [2. 🔹 Создание таблиц](#2--создание-таблиц)
      - [3. 🔹 Основные типы данных](#3--основные-типы-данных)
      - [4. 🔹 Модификация таблиц](#4--модификация-таблиц)
      - [5. 🔹 Индексы](#5--индексы)
      - [6. 🔹 SELECT и работа с данными](#6--select-и-работа-с-данными)
        - [🔹 SELECT — выборка данных](#-select--выборка-данных)
        - [🔹 JOIN — объединение таблиц](#-join--объединение-таблиц)
        - [🔹 INSERT — добавление данных](#-insert--добавление-данных)
        - [🔹 UPDATE — обновление данных](#-update--обновление-данных)
        - [🔹 DELETE — удаление данных](#-delete--удаление-данных)
      - [7. 🔹 Транзакции](#7--транзакции)

## Databases core

### Основы работы с БД

#### 1. 🔹 Создание базы данных
```
-- Создать новую БД
CREATE DATABASE shop;

-- Использовать БД (PostgreSQL)
\c shop;

-- Использовать БД (MySQL)
USE shop;
```

#### 2. 🔹 Создание таблиц
```
CREATE TABLE users (
    id SERIAL PRIMARY KEY,        -- автоинкремент
    name VARCHAR(100) NOT NULL,   -- строка до 100 символов
    age INT CHECK (age > 0),      -- число с ограничением
    email VARCHAR(150) UNIQUE,    -- уникальное поле
    created_at TIMESTAMP DEFAULT NOW()
);
```

Пример второй таблицы:
```
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),   -- внешний ключ
    amount DECIMAL(10,2) NOT NULL,      -- деньги
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### 3. 🔹 Основные типы данных

**Числовые**

- `INT` / `INTEGER` — целые числа
- `BIGINT` — большие целые
- `SERIAL` — автоинкремент (PostgreSQL)
- `DECIMAL(precision, scale)` — точные числа (например, для денег)
- `FLOAT`, `REAL` — числа с плавающей точкой

**Строковые**

- `CHAR(n)` — строка фиксированной длины
- `VARCHAR(n)` — строка до n символов
- `TEXT` — длинный текст

**Даты и время**

- `DATE` — только дата
- `TIME` — только время
- `TIMESTAMP` — дата и время
- `NOW()` — текущие дата и время

**Логические**

- `BOOLEAN` — true/false

**Прочее**

- `UUID` — универсальный идентификатор
- `JSON` / `JSONB` (PostgreSQL) — хранение JSON

#### 4. 🔹 Модификация таблиц
```
-- Добавить колонку
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Удалить колонку
ALTER TABLE users DROP COLUMN phone;

-- Изменить тип
ALTER TABLE users ALTER COLUMN age TYPE SMALLINT;
```

#### 5. 🔹 Индексы

Ускоряют поиск, особенно по фильтрам и JOIN.
```
-- Индекс по email
CREATE INDEX idx_users_email ON users(email);

-- Уникальный индекс
CREATE UNIQUE INDEX uniq_users_name ON users(name);
```

#### 6. 🔹 SELECT и работа с данными

##### 🔹 SELECT — выборка данных

- Основная команда для получения данных из таблицы.

**Синтаксис:**
```
SELECT столбцы
FROM таблица
WHERE условие
ORDER BY поле [ASC|DESC]
LIMIT n OFFSET m;
```

**Примеры:**
```
-- Все колонки
SELECT * FROM users;

-- Определённые колонки
SELECT id, name FROM users;

-- С фильтром
SELECT id, name FROM users WHERE age > 18;

-- С сортировкой
SELECT id, name FROM users ORDER BY age DESC;

-- С пагинацией
SELECT id, name FROM users LIMIT 10 OFFSET 20;
```

##### 🔹 JOIN — объединение таблиц

Используется для связи данных из разных таблиц.

**Виды JOIN:**

1. **INNER JOIN** — только совпадения.
2. **LEFT JOIN** — все из левой + совпадения из правой.
3. **RIGHT JOIN** — все из правой + совпадения из левой.
4. **FULL JOIN** — все строки обеих таблиц.

**Примеры:**
```
-- INNER JOIN
SELECT u.id, u.name, o.amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN
SELECT u.id, u.name, o.amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- RIGHT JOIN
SELECT u.id, u.name, o.amount
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;

-- FULL JOIN (не всегда поддерживается в MySQL)
SELECT u.id, u.name, o.amount
FROM users u
FULL JOIN orders o ON u.id = o.user_id;
```

##### 🔹 INSERT — добавление данных

Добавляет строки в таблицу.

**Синтаксис:**
```
INSERT INTO таблица (колонки) VALUES (значения);
```

**Примеры:**
```
-- Один пользователь
INSERT INTO users (name, age) VALUES ('Alice', 25);

-- Несколько пользователей
INSERT INTO users (name, age) VALUES ('Bob', 30), ('Charlie', 28);
```

##### 🔹 UPDATE — обновление данных

Изменяет существующие записи.

**Синтаксис:**
```
UPDATE таблица SET поле = значение WHERE условие;
```

⚠️ Без WHERE обновятся все строки.

**Примеры:**
```
-- Изменить возраст
UPDATE users SET age = 26 WHERE name = 'Alice';

-- Увеличить возраст всем
UPDATE users SET age = age + 1;
```

##### 🔹 DELETE — удаление данных

Удаляет строки из таблицы.

**Синтаксис:**
```
DELETE FROM таблица WHERE условие;
```

**Примеры:**
```
-- Удалить пользователя
DELETE FROM users WHERE id = 1;

-- Удалить всех
DELETE FROM users;
```

#### 7. 🔹 Транзакции

Транзакция = набор SQL-операций, которые выполняются **атомарно**.
Либо все изменения применяются (COMMIT), либо ни одно (ROLLBACK).

**Синтаксис:**
```
BEGIN;            -- начало транзакции
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;           -- фиксация
-- ROLLBACK;      -- отмена (если ошибка)
```

**Пример с переводом денег:**

```
BEGIN;

UPDATE accounts SET balance = balance - 200 WHERE id = 1;
UPDATE accounts SET balance = balance + 200 WHERE id = 2;

COMMIT;
```