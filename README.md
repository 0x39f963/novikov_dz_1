# Домашнее задание №1  
Новиков Иван

Это обновленный отчет по ДЗ (форматирование, корерктно описание). 

---

## 1. Развёртывание PostgreSQL и pgAdmin

PostgreSQL и pgAdmin были развернуты в Docker под WSL2 
Созданы постоянные тома, чтобы данные не терялись при перезагрузке

Скриншоты (они чуть размыты, но view'р яндекса позволяет скачать файл - и скриншот будет в номральном качестве):
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
Скрин: https://disk.yandex.ru/i/qr0qCWrgM5pzGw

### Таблица products

CREATE TABLE products (
    product_id int PRIMARY KEY,
    brand varchar(100) NOT NULL,
    product_line varchar(50),
    product_class varchar(50),
    product_size varchar(50),
    list_price numeric(10,2) NOT NULL,
    standard_cost numeric(10,2) NOT NULL
);
Скрин: https://disk.yandex.ru/i/3rm-8L3prAp-fg

### Таблица transactions

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

https://disk.yandex.ru/i/dKXrWskRyIGhkg

https://disk.yandex.ru/i/kSQVYkOd48MhDg

