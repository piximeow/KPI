---
status: accepted
date: 2026-11-24
---

# HTTP-вхід у систему — окремі компоненти `CheckoutAPI` і `PaymentWebhook`

## Context and Problem Statement

У lab3 `CheckoutUI` кличе `CheckoutService` напряму, а Платіжний шлюз — `PaymentService`. На компонентній діаграмі `CheckoutUI` працює в браузері Клієнта, а сервіси — на сервері; шлюз — зовнішня система за межею. Тож обидва кличуть серверну частину по мережі, лише HTTPS (NFR-05). Що саме приймає ці виклики на сервері?

## Considered Options

* A. Окремі компоненти HTTP-входу: `CheckoutAPI` (виклики UI) і `PaymentWebhook` (повідомлення шлюзу); сервіси лишаються як у lab3.
* B. Сервіси самі надають HTTP-інтерфейси браузеру й шлюзу, без проміжного шару.

## Decision Outcome

Chosen option: A, бо мережею до сервера доступні лише інтерфейси API (`ICheckout` для браузера, `IPaymentNotify` для шлюзу), а сервіси й таймер лишаються за ними, з тими самими операціями, що в lab3. Компоненти API не мають власної логіки: вони лише приймають HTTP-запит і передають ту саму операцію (`openCheckout`, `confirmOrder`, `notifyPayment`) сервісу, тож рішення lab3 не змінюються.

### Consequences

* Good, бо `onPaymentTimeout` недосяжний ні з браузера, ні зі шлюзу: `IPaymentEvents` кличуть лише `PaymentWebhook` і `PaymentTimer`.
* Good, бо шари діаграми однозначні: UI → API → сервіси → Data Access → БД.
* Bad, бо в lab3 цих компонентів немає, а в lab4 вони не виділені з доріжки «Магазин» — в `audit.md` позначені як нові.

## Pros and Cons of the Options

### A. Окремий HTTP-вхід
* Good, бо мережею до сервера доступні лише два інтерфейси API.
* Bad, бо на два компоненти більше, ніж учасників lab3.

### B. Сервіси напряму
* Good, бо сервіси й таймер, як у lab3, — без проміжного шару.
* Bad, бо `IPaymentEvents` з `onPaymentTimeout` став би доступним шлюзу по мережі.