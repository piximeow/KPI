# Специфікація: ER-модель «Онлайн-магазин»

Мета: описати дані онлайн-магазину як концептуальну ER-модель (діаграма Mermaid). Без SQL.

## Правила

- Усі моделі в 3НФ; похідні значення (сума замовлення) не зберігаються.
- Усі `*_id` мають тип `int` (ціле додатне число). Первинний ключ називається `<сутність>_id`, зовнішній ключ на неї скрізь називається так само.
- Поза ключами назви атрибутів не повторюються (`category_name`, `product_name`, а не `name`). Назви: англійською, `snake_case`, однина.
- Зв'язок M:N показується напряму, без сполучної таблиці.
- Обмеження (діапазони, значення статусів, необов'язкові поля, унікальні пари) обов'язково вказуються в діаграмі коментарями до атрибутів.
- Типи: `int` — ціле число; `string` — текст; `decimal` — число з двома знаками після коми; `datetime` — дата й час.
- Позначки: PK — первинний ключ, FK — зовнішній, UQ — унікальне значення (в діаграмі пишеться як UK).

## Сутності

| Сутність | Атрибути |
|----------|----------|
| customer | **customer_id** (int, PK), first_name (string), last_name (string), email (string, UQ), phone (string, необов'язковий), registered_at (datetime) |
| address | **address_id** (int, PK), customer_id (int, FK), country (string), city (string), street_line (string), postal_code (string) |
| category | **category_id** (int, PK), category_name (string, UQ) |
| product | **product_id** (int, PK), product_name (string), sku (string, UQ), description (string, необов'язковий), price (decimal, ≥ 0), stock_quantity (int, ≥ 0) |
| order | **order_id** (int, PK), address_id (int, FK), order_status (string: `new`, `paid`, `shipped`, `delivered`, `cancelled`), ordered_at (datetime) |
| order_item | **order_item_id** (int, PK), order_id (int, FK), product_id (int, FK, унікальна пара з order_id), quantity (int, ≥ 1), unit_price (decimal, ≥ 0, ціна на момент замовлення) |
| payment | **payment_id** (int, PK), order_id (int, FK), payment_method (string: `card`, `cash_on_delivery`, `bank_transfer`), payment_status (string: `pending`, `succeeded`, `failed`, `refunded`), payment_amount (decimal, > 0), paid_at (datetime, необов'язковий) |
| review | **review_id** (int, PK), customer_id (int, FK), product_id (int, FK, унікальна пара з customer_id), rating (int, 1–5), review_text (string, необов'язковий), reviewed_at (datetime) |

## Зв'язки

| № | Зв'язок | Назва | Кардинальність | Участь |
|---|---------|-------|----------------|--------|
| L1 | customer — address | `has` | 1 : N | клієнт має 0..N адрес; адреса належить рівно 1 клієнту |
| L2 | address — order | `receives` | 1 : N | на адресу 0..N замовлень; замовлення йде на рівно 1 адресу (клієнт замовлення визначається через адресу) |
| L3 | category — product | `contains` | **M : N** | категорія містить 0..N товарів; товар у 0..N категорій |
| L4 | order — order_item | `includes` | 1 : N | замовлення має 1..N позицій; позиція належить рівно 1 замовленню |
| L5 | product — order_item | `listed in` | 1 : N | товар у 0..N позицій; позиція стосується рівно 1 товару |
| L6 | order — payment | `paid_by` | 1 : N | замовлення має 0..N платежів; платіж стосується рівно 1 замовлення |
| L7 | customer — review | `writes` | 1 : N | клієнт пише 0..N відгуків; відгук має рівно 1 автора |
| L8 | product — review | `has` | 1 : N | товар має 0..N відгуків; відгук стосується рівно 1 товару |

## Критерії прийняття

- [ ] 1. У моделі рівно 8 сутностей: customer, address, category, product, order, order_item, payment, review.
- [ ] 2. Усі 8 зв'язків (L1–L8) показані з назвою, кардинальністю та участю; на діаграмі немає нічого, чого немає тут.
- [ ] 3. Кожна сутність має PK `<сутність>_id`; усі `*_id` мають тип `int`.
- [ ] 4. Кожен FK має ту саму назву, що й PK, на який посилається (`customer_id`, `order_id` тощо).
- [ ] 5. Поза ключами жодна назва атрибута не повторюється в різних сутностях.
- [ ] 6. Зв'язок category — product показаний як прямий M:N: у `product` немає `category_id`, сполучної сутності немає.
- [ ] 7. Товари замовлення зв'язані лише через `order_item` (1:N з `order` і 1:N з `product`); `quantity` і `unit_price` лежать у `order_item`.
- [ ] 8. У `order` немає `customer_id`: клієнт визначається через `address`.
- [ ] 9. Кожна сутність у 3НФ, похідних атрибутів немає.
- [ ] 10. Кожен атрибут має тип зі списку типів; діапазони, значення статусів, необов'язкові поля й унікальні пари з таблиці сутностей показані в діаграмі коментарями, UQ — позначкою UK.