## ДЗ-02

### Схема

![SchemeDB](./scheme.png)

### Описание таблиц и полей

```sql
CREATE TABLE categories
(
    id          INTEGER
        primary key autoincrement,
    name        VARCHAR(255) not null,
    description TEXT
);

CREATE TABLE contacts
(
    id              INTEGER
        primary key autoincrement,
    first_name      VARCHAR(255) not null,
    middle_name     VARCHAR(255) not null,
    last_name       VARCHAR(255) not null,
    birth_date      DATE         not null,
    residence_place text,
    extra_info      text
);

CREATE TABLE customers
(
    id         INTEGER
        primary key autoincrement,
    name       VARCHAR(255) not null,
    contact_id int
        references contacts,
    extra_info text
);

CREATE TABLE manufacturers
(
    id         INTEGER
        primary key autoincrement,
    name       VARCHAR(255) not null,
    contact_id int
        references contacts,
    extra_info text
);

CREATE TABLE purchases (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    customer_id INT,
    paid_date DATE NOT NULL,
    quantity INT NOT NULL,
    total_price DECIMAL NOT NULL,
    CONSTRAINT FK_purchases_customers FOREIGN KEY (customer_id) REFERENCES customers(id)
);

CREATE TABLE suppliers
(
    id         INTEGER
        primary key autoincrement,
    name       VARCHAR(255) not null,
    contact_id int
        references contacts,
    extra_info text
);

CREATE TABLE products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR NOT NULL,
    category_id INT,
    manufacturer_id INT,
    supplier_id INT,
    description TEXT,
    CONSTRAINT FK_products_manufacturers_2 FOREIGN KEY (manufacturer_id) REFERENCES manufacturers(id),
    CONSTRAINT FK_products_suppliers_3 FOREIGN KEY (supplier_id) REFERENCES suppliers(id)
);

CREATE TABLE prices
(
    id         INTEGER
        primary key autoincrement,
    product_id INT
        references products,
    price      DECIMAL(10, 2) not null,
    valid_from DATE,
    valid_to   DATE
);

CREATE TABLE product_category (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    product_id INT REFERENCES products,
    category_id INT REFERENCES categories
);

CREATE TABLE product_purchase (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    product_id INT REFERENCES products,
    purchase_id INT REFERENCES purchases
);
```

1. Анализ возможных запросов/отчетов/поиска данных
```
Запросы по категориям:
   Получение всех категорий.
   Получение категории по имени.

Запросы по контактам:
   Поиск контактов по имени или фамилии.
   Получение всех контактов, связанных с покупателями, производителями и поставщиками.

Запросы по покупателям:
   Получение всех покупателей.
   Поиск покупателя по идентификатору контакта.

Запросы по производителям и поставщикам:
   Получение всех производителей и поставщиков.
   Поиск по имени.

Запросы по продуктам:
   Получение всех продуктов.
   Поиск продукта по имени или категории.
   Получение всех продуктов от конкретного производителя или поставщика.

Запросы по покупкам:
   Получение всех покупок.
   Поиск покупок по покупателю или дате.
   Получение всех покупок конкретного продукта.

Запросы по ценам:
   Получение всех цен для продукта.
   Поиск цен по дате.
```

2. Предполагаем возможную кардинальность поля

   `categories.name`: низкая (может быть много категорий, но названия могут повторяться).

   `contacts.first_name`, `last_name`: высокая (много уникальных имен и фамилий).

   `customers.name`: высокая (много уникальных имен).

   `manufacturers.name`: высокая (много уникальных производителей).

   `products.name`: высокая (много уникальных продуктов).

   `purchases.customer_id`: средняя (много покупок, при ограниченном количестве покупателей).

   `prices.product_id`: высокая (много цен для разных продуктов).


3. Создаем дополнительные индексы
```sql
-- Индексы для ускорения поиска
CREATE INDEX idx_contacts_last_name ON contacts(last_name);
-- Ускоряет поиск контактов по фамилии

CREATE INDEX idx_contacts_first_name ON contacts(first_name);
-- Ускоряет поиск контактов по имени

CREATE INDEX idx_customers_contact_id ON customers(contact_id);
-- Ускоряет поиск покупателей по идентификатору контакта

CREATE INDEX idx_products_name ON products(name);
-- Ускоряет поиск продуктов по имени

CREATE INDEX idx_purchases_customer_id ON purchases(customer_id);
-- Ускоряет поиск покупок по идентификатору покупателя

CREATE INDEX idx_prices_product_id ON prices(product_id);
-- Ускоряет поиск цен по идентификатору продукта

CREATE INDEX idx_product_category_product_id ON product_category(product_id);
-- Ускоряет поиск категорий по идентификатору продукта

CREATE INDEX idx_product_purchase_product_id ON product_purchase(product_id);
-- Ускоряет поиск покупок по идентификатору продукта
```

4. Описание индексов

    `idx_contacts_last_name`: Ускоряет поиск контактов по фамилии, что часто используется в отчетах и запросах.

    `idx_contacts_first_name`: Ускоряет поиск контактов по имени, что также часто используется.

    `idx_customers_contact_id`: Ускоряет поиск покупателей по идентификатору контакта, что важно для связи между таблицами.

    `idx_products_name`: Ускоряет поиск продуктов по имени, что важно для быстрого доступа к информации о продукте.

    `idx_purchases_customer_id`: Ускоряет поиск покупок по идентификатору покупателя, что важно для анализа покупок.

    `idx_prices_product_id`: Ускоряет поиск цен по идентификатору продукта, что важно для получения актуальной информации о ценах.

    `idx_product_category_product_id`: Ускоряет поиск категорий по идентификатору продукта, что важно для анализа категорий продуктов.

    `idx_product_purchase_product_id`: Ускоряет поиск покупок по идентификатору продукта, что важно для анализа продаж.

5. Логические ограничения в БД

    * Уникальность:

   `UNIQUE(name)` в таблицах `categories`, `customers`, `manufacturers`, `products` для предотвращения дублирования.
  
   В таблице связей `product_category` уникальный набор `unique(product_id, category_id)` для предотвращения дублирования связей между товаром и категорией.

   В таблице связей `product_purchase` уникальный набор `unique(product_id, purchase_id)` для предотвращения дублирования связей между товаром и покупкой.

6. Ограничения по выбранным полям

    * Проверка на отрицательные значения:
   
    В таблице `purchases` необходимо проверка на поле `quantity` (количество) и `total_price` (общую стоимость, равно 0 - при акциях\скидках\подарках), чтобы они не были отрицательными.

    В таблице `prices` проверка поля `price` (цена) на значение не отрицательное.
