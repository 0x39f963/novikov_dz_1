Домашнее задание №1, Новиков Иван

1. Развертывание постгресс и pgAdmin
https://disk.yandex.ru/i/6ov1ln0_CBMBTg
https://disk.yandex.ru/i/kdsHeVQZgkVA9A 



2. Критерий 1. Продумать структуру базы данных и отрисовать в редакторе
https://disk.yandex.ru/i/Nin513nwow4gcg


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


https://disk.yandex.ru/i/qr0qCWrgM5pzGw

CREATE TABLE products (
    product_id int PRIMARY KEY,

    brand varchar(100) NOT NULL,
    product_line varchar(50),
    product_class varchar(50),
    product_size varchar(50),

    list_price numeric(10,2) NOT NULL,
    standard_cost numeric(10,2) NOT NULL
);

https://disk.yandex.ru/i/3rm-8L3prAp-fg

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

https://disk.yandex.ru/i/Bt8oWrnaMJh_1g


https://disk.yandex.ru/i/fEVGQcDIl76wbw

csv данные
https://disk.yandex.ru/i/UMX4SDatWQt9qg

https://disk.yandex.ru/i/IJBF9HiKhNg1xg



https://disk.yandex.ru/i/dKXrWskRyIGhkg

\\wsl$\Ubuntu-24.04\home\x39963\mipt_pg


https://disk.yandex.ru/i/kSQVYkOd48MhDg
