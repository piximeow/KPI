# Специфікація: ER-модель «Онлайн-магазин»

Мета: описати дані онлайн-магазину як концептуальну ER-модель (діаграма буде наступним кроком). Без SQL.

## Правила

- Усі моделі в 3НФ; похідні значення (сума замовлення) не зберігаються.
- Усі `*_id` мають один тип: ціле додатне число. Первинний ключ називається `<сутність>_id`, зовнішній ключ на неї скрізь називається так само.
- Поза ключами назви атрибутів не повторюються (`category_name`, `product_name`, а не `name`). Назви: англійською, `snake_case`, однина.
- Зв'язок M:N показується напряму, без сполучної таблиці.

## Сутності

PK — первинний ключ, FK — зовнішній, UQ — унікальне значення. Текст, число з копійками, ціле, дата й час — типи за змістом.

| Сутність | Атрибути |
|----------|----------|
| customer | **customer_id** (PK), first_name, last_name, email (UQ), phone (необов'язковий), registered_at |
| address | **address_id** (PK), customer_id (FK), country, city, street_line, postal_code |
| category | **category_id** (PK), category_name (UQ) |
| product | **product_id** (PK), product_name, sku (UQ), description (необов'язковий), price (≥ 0), stock_quantity (≥ 0) |
| order | **order_id** (PK), address_id (FK), order_status (`new`, `paid`, `shipped`, `delivered`, `cancelled`), ordered_at |
| order_item | **order_item_id** (PK), order_id (FK), product_id (FK), quantity (≥ 1), unit_price (≥ 0, ціна на момент замовлення) |
| payment | **payment_id** (PK), order_id (FK), payment_method (`card`, `cash_on_delivery`, `bank_transfer`), payment_status (`pending`, `succeeded`, `failed`, `refunded`), payment_amount (> 0), paid_at (необов'язковий) |
| review | **review_id** (PK), customer_id (FK), product_id (FK), rating (1–5), review_text (необов'язковий), reviewed_at |

## Зв'язки

| № | Зв'язок | Кардинальність | Участь |
|---|---------|----------------|--------|
| L1 | customer — address | 1 : N | клієнт має 0..N адрес; адреса належить рівно 1 клієнту |
| L2 | address — order | 1 : N | на адресу 0..N замовлень; замовлення йде на рівно 1 адресу (клієнт замовлення визначається через адресу) |
| L3 | category — product | **M : N** | категорія містить 0..N товарів; товар у 0..N категорій |
| L4 | order — order_item | 1 : N | замовлення має 1..N позицій; позиція належить рівно 1 замовленню |
| L5 | product — order_item | 1 : N | товар у 0..N позицій; позиція стосується рівно 1 товару |
| L6 | order — payment | 1 : N | замовлення має 0..N платежів; платіж стосується рівно 1 замовлення |
| L7 | customer — review | 1 : N | клієнт пише 0..N відгуків; відгук має рівно 1 автора |
| L8 | product — review | 1 : N | товар має 0..N відгуків; відгук стосується рівно 1 товару |

## Критерії прийняття

- [ ] 1. У моделі рівно 8 сутностей: customer, address, category, product, order, order_item, payment, review.
- [ ] 2. Усі 8 зв'язків (L1–L8) показані з кардинальністю та участю; на діаграмі немає нічого, чого немає тут.
- [ ] 3. Кожна сутність має PK `<сутність>_id`; усі `*_id` одного типу (ціле додатне).
- [ ] 4. Кожен FK має ту саму назву, що й PK, на який посилається (`customer_id`, `order_id` тощо).
- [ ] 5. Поза ключами жодна назва атрибута не повторюється в різних сутностях.
- [ ] 6. Зв'язок category — product показаний як прямий M:N: у `product` немає `category_id`, сполучної сутності немає.
- [ ] 7. Товари замовлення зв'язані лише через `order_item` (1:N з `order` і 1:N з `product`); `quantity` і `unit_price` лежать у `order_item`.
- [ ] 8. У `order` немає `customer_id`: клієнт визначається через `address`.
- [ ] 9. Кожна сутність у 3НФ, похідних атрибутів немає.
- [ ] 10. Обмеження вказані: UQ для `email`, `sku`, `category_name`; унікальна пара (`order_id`, `product_id`) в `order_item`; один відгук на пару (`customer_id`, `product_id`); діапазони для `price`, `stock_quantity`, `quantity`, `unit_price`, `payment_amount`, `rating`.