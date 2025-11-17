# Домашнее задание №1  
Новиков Иван

Это обновленный отчет по ДЗ (форматирование, корректное описание). 

---

## 1. Развёртывание PostgreSQL и pgAdmin

PostgreSQL и pgAdmin были развернуты в Docker под WSL2 
Созданы постоянные тома, чтобы данные не терялись при перезагрузке

Скриншоты (они чуть размыты, но view'ер яндекса позволяет скачать файл - и скриншот будет в номральном качестве) :
- https://disk.yandex.ru/i/6ov1ln0_CBMBTg  
- https://disk.yandex.ru/i/kdsHeVQZgkVA9A

---

## 2. Структура базы данных и диаграмма

Определены три сущности: 
- customers  
- products  
- transactions  

Между ними настроены связи многие к одному

Диаграмма:
- https://disk.yandex.ru/i/Nin513nwow4gcg

**upd:** 
в ходе решения 5 части и иправления того, что получислось изначально, 
итоговая схема будет https://disk.yandex.ru/i/FWWNIyBPtnd7Iw 

Подробности со всеми скриншотами в части 5.

---

## 3. Нормализация

В исходных дланных - 2 листа.

### Приведение к 1НФ
1НФ требует, чтобы в таблице:
- не было повторяющихся групп значений,
- каждый столбец был атомарным.

В исходных данных поля товара и поля клиента смешаны в одной строке сделки, соотв. я выделилл отдельные сущности:
товары, клиенты, транзакции

### Приведение ко 2НФ
2НФ требует отсутствия частичных зависимостей от составного ключа.  
В исходной таблице ключом является transaction_id,но данные о товаре и клиенте **не зависят** от transaction_id - это нарушение 2НФ.

В итоге я данные клиентов вынесены в таблицу customers, данные товаров выгес в таблицу products.

### Приведение к 3НФ
3НФ запрещает транзитивные зависимости.  
В исходной таблице значения вроде brand, product_line и т.п. зависят не от самой транзакции, а от product_id.  
Это нарушение 3НФ.

Соотв.:  
- атрибуты товара вынесены в products,  
- транзакции хранят только ссылки (product_id, customer_id).

В итоге структура приведена к 3НФ: каждая таблица описывает ровно одну сущность, а связи выражены через внешние ключи.

---

## 4. Создание таблиц

### Таблица customers
```sql
CREATE TABLE customers (
    customer_id int PRIMARY KEY,
    first_name varchar(100) NOT NULL,
    last_name varchar(100),
    gender varchar(20),
    dob date,
    job_title varchar(200),
    job_industry_category varchar(200),
    wealth_segment varchar(50),
    deceased_indicator varchar(5),
    owns_car varchar(5),
    address varchar(255),
    postcode int,
    state varchar(50),
    country varchar(100),
    property_valuation int
);
```
Скрин: https://disk.yandex.ru/i/qr0qCWrgM5pzGw

### Таблица products
```sql
CREATE TABLE products (
    product_id int PRIMARY KEY,
    brand varchar(100) NOT NULL,
    product_line varchar(50),
    product_class varchar(50),
    product_size varchar(50),
    list_price numeric(10,2) NOT NULL,
    standard_cost numeric(10,2) NOT NULL
);
```
Скрин: https://disk.yandex.ru/i/3rm-8L3prAp-fg

### Таблица transactions
```sql
CREATE TABLE transactions (
    transaction_id int PRIMARY KEY,
    product_id int NOT NULL,
    customer_id int NOT NULL,
    transaction_date date NOT NULL,
    online_order boolean,
    order_status varchar(50),
    CONSTRAINT fk_transactions_product
        FOREIGN KEY (product_id) REFERENCES products(product_id),
    CONSTRAINT fk_transactions_customer
        FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

Скрин:

https://disk.yandex.ru/i/Bt8oWrnaMJh_1g

### Созданные таблицы:
Скрин:
https://disk.yandex.ru/i/fEVGQcDIl76wbw

## 5. Импорт данных из CSV
Данные были импортированы через pgAdmin:

CSV данные:

https://disk.yandex.ru/i/UMX4SDatWQt9qg

https://disk.yandex.ru/i/IJBF9HiKhNg1xg

\\wsl$\Ubuntu-24.04\home\x39963\mipt_pg

Скриншоты импорта:

https://disk.yandex.ru/i/uFxzdBTRpBy3rg 

https://disk.yandex.ru/i/dKXrWskRyIGhkg

https://disk.yandex.ru/i/kSQVYkOd48MhDg

https://disk.yandex.ru/i/KupRixtHeeaggg - запятые на точки поменял руками в гуглдокс.

### В процессе импорта обнаружил битые данные,
Пример битых данных:
https://disk.yandex.ru/i/0uHtikX_OsPF3Q -  сначала думал это точечные проблемы... 

https://disk.yandex.ru/i/caEXiC7-z2izww - 6 стадий принятия :)

### Epic Fail: я понял, что product_id абсолютно не уникален. 

**Придется все переделать :) lol**

Я не уверен до конца, специально так планировалось, что мы должен решить эту проблему, или просто данные битые.
НЕ совсем ясно, какое решение от нас ожидали. Мне на ум приходит собрать строки из всех параметров товара и наделать из них md5 ключей, а потом этим строкам
выдать новые (условно product_id_uniq)

**в итоге: **

1. делаю таблицу сырых данных -> гружусь в нее

```sql
CREATE TABLE raw_transactions (
    transaction_id int,
    product_id int,
    customer_id int,
    transaction_date date,
    online_order boolean,
    order_status varchar(50),
    brand varchar(100),
    product_line varchar(50),
    product_class varchar(50),
    product_size varchar(50),
    list_price numeric,
    standard_cost numeric
);
```

Скрин:
https://disk.yandex.ru/i/h2DNvfIoVqnUFQ 

Импортирую все данные:
https://disk.yandex.ru/i/OhEWde1KrKeizw

2. делаю доп поле, чтобы туда загнать хеши:

https://disk.yandex.ru/i/63d4eMF0-tQT7w 

тут пришлось погуглить (coalesce для null значений, ::text для перевода чисел в строки)

```sql
UPDATE raw_transactions
SET product_hash_uid = md5(
    coalesce(brand, '') ||
    coalesce(product_line, '') ||
    coalesce(product_class, '') ||
    coalesce(product_size, '') ||
    coalesce(list_price::text, '') ||
    coalesce(standard_cost::text, '')
);
```

Скрин:
https://disk.yandex.ru/i/iSlVTnS6_bGyEA

Вроде все ОК: 
https://disk.yandex.ru/i/0jE0pdLpt3unIA

3. Собираем новую таблицу

```sql
CREATE TABLE products_clean (
    product_uid varchar(32) PRIMARY KEY,
    brand varchar(100),
    product_line varchar(50),
    product_class varchar(50),
    product_size varchar(50),
    list_price numeric(10,2),
    standard_cost numeric(10,2)
);
```

Скрин:
https://disk.yandex.ru/i/fkqZdbkup_uZaQ

и заполняем ее уник строками товаров:

```sql
INSERT INTO products_clean (
    product_uid,
    brand,
    product_line,
    product_class,
    product_size,
    list_price,
    standard_cost
)
SELECT DISTINCT
    product_hash_uid AS product_uid,
    brand,
    product_line,
    product_class,
    product_size,
    list_price,
    standard_cost
FROM raw_transactions;
```

Скрин:
https://disk.yandex.ru/i/RUSbITOcfwIk7Q

данные занесены в таблицу:
https://disk.yandex.ru/i/zw7nyTZ7tJCzoQ

4. Собираем таблицу транзакций, связанную с новой таблицей продуктов:

Скрин:
https://disk.yandex.ru/i/C2o5bPolso1qSA 

```sql
CREATE TABLE transactions_clean (
    transaction_id int PRIMARY KEY,

    product_uid varchar(32) NOT NULL,
    customer_id int NOT NULL,

    transaction_date date NOT NULL,

    online_order boolean,
    order_status varchar(50),

    FOREIGN KEY (product_uid) REFERENCES products_clean(product_uid),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

наполняем ее :

*Данные оказались чуть битые:*  https://disk.yandex.ru/i/UwZIZPjU35pKIg 

Всего таких строк оказалось три: https://disk.yandex.ru/i/ezCaAsV9nGDaXw 

**Поправил их руками:**

```sql
UPDATE raw_transactions
SET customer_id = 1
WHERE customer_id = 5034;
```
Скрин: https://disk.yandex.ru/i/ZR9Efda6S_2VAg


Собираем таблицу:

```sql
INSERT INTO transactions_clean (
    transaction_id,
    product_uid,
    customer_id,
    transaction_date,
    online_order,
    order_status
)
SELECT
    transaction_id,
    product_hash_uid AS product_uid,
    customer_id,
    transaction_date,
    online_order,
    order_status
FROM raw_transactions;
```

Скрин (всего 20 000 строк, как и было в сырых данных, до перелопачивания :)
https://disk.yandex.ru/i/j8b1LIyvEzHGWg


Теперь формат данных норм.

## Обновленная схема:

https://disk.yandex.ru/i/FWWNIyBPtnd7Iw