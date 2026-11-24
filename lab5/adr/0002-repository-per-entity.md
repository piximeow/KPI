---
status: accepted
date: 2026-11-24
---

# Data Access — репозиторій на кожного учасника-сутність lab3, `order` і `order_item` в одному

## Context and Problem Statement

У lab3 сервіси звертаються до сутностей lab1 як до окремих учасників: `address`, `product`, `order`, `order_item`, `payment`. До `customer` звернень немає — він лише ціль FK `address.customer_id` і параметр `findByCustomer`. Як поділити шар Data Access?

## Considered Options

* A. Репозиторій на кожного учасника-сутність lab3, `order` і `order_item` — в одному `OrderRepository`.
* B. Окремий репозиторій на кожну з п'яти сутностей.
* C. Один репозиторій на всі таблиці.

## Decision Outcome

Chosen option: A, бо `order_item` в UC2 не існує без свого `order`: позиції вставляються в тій самій транзакції, що й `order` (S01, S02), і читаються лише за ним (`findByOrder`). Операції інтерфейсів — ті самі, що в lab3; `customer` репозиторію не має, бо в lab3 до нього не звертаються, а таблиця є в `ShopDB` як ціль FK.

### Consequences

* Good, бо видно, що саме кожен сервіс потребує від даних: `PaymentService` не торкається `IAddresses`.
* Good, бо кожна операція `IOrders` — з учасника `order` або `order_item` у lab3.
* Bad, бо `OrderRepository` відповідає двом учасникам lab3 — це записано в таблиці відповідності `audit.md`.

## Pros and Cons of the Options

### A. Репозиторій на сутність, `order` з `order_item`
* Good, бо межі репозиторіїв збігаються з межами транзакцій lab3.
* Bad, бо не один в один з учасниками lab3.

### B. П'ять репозиторіїв
* Good, бо один в один з учасниками lab3.
* Bad, бо `CheckoutService` і `PaymentService` отримують ще одну залежність заради позицій, які без `order` не мають сенсу.

### C. Один репозиторій
* Good, бо найменше компонентів.
* Bad, бо не видно, які дані потрібні кожному сервісу, і `PaymentService` формально отримує доступ до адрес.