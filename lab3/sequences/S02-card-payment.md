# S02. Клієнт оплачує замовлення карткою

Сценарій: `@S2` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги сценарію: REQ-03, REQ-04, REQ-10, REQ-11 · також: REQ-01, REQ-05, REQ-06, REQ-07, REQ-08, REQ-09, REQ-13, REQ-16, REQ-17, REQ-19, REQ-20, REQ-21 · кроки інтерфейсу поза lab2: disableConfirm(), кроки шлюзу: Клієнт вводить дані картки, шлюз показує сторінку результату (UC5, поза lab2) · рендер: [`svg/S02.svg`](svg/S02.svg)

```mermaid
sequenceDiagram
    autonumber
    actor C as Клієнт
    participant UI as CheckoutUI
    participant CS as CheckoutService
    participant AD as address
    participant PR as product
    participant OR as order
    participant OI as order_item
    participant PA as payment
    participant TM as PaymentTimer
    actor GW as Платіжний шлюз
    participant PS as PaymentService

    Note over C,PS: Дано: Клієнт має address у Києві, у кошику 2 шт. «Навушники» (price 1200.00, stock_quantity 5)
    C->>+UI: відкриває форму («Оформити»)
    UI->>+CS: openCheckout(customer_id)
    CS->>+AD: findByCustomer(customer_id)
    AD-->>-CS: [address «Київ»]
    CS-->>-UI: форма з address «Київ», card, cash_on_delivery (REQ-01)
    UI-->>-C: форма
    C->>+UI: обирає address «Київ» і card, натискає «Підтвердити»
    UI->>UI: disableConfirm()
    UI->>+CS: confirmOrder(customer_id, address_id, payment_method = card, кошик)
    CS->>+OR: countOrderedSince(ordered_at ≥ now − 60 с)
    OR-->>-CS: кількість order
    CS->>CS: checkForm()
    Note over CS: REQ-20, REQ-19, REQ-08 не виконуються — як у S01
    critical транзакція REQ-03–REQ-05 (REQ-06)
        CS->>+PR: lockForUpdate(product_id кожного товару кошика)
        PR-->>-CS: price = 1200.00, stock_quantity = 5
        CS->>CS: checkStockAndTotal() — REQ-07, REQ-09 не виконуються
        CS->>+OR: insertOrder(order_status = new, ordered_at = now, address_id)
        OR-->>-CS: order_id
        loop кожен product кошика
            CS->>+OI: insertOrderItem(order_id, product_id, quantity = 2, unit_price = 1200.00)
            OI-->>-CS: order_item_id
            CS->>+PR: decreaseStock(product_id, quantity = 2)
            PR-->>-CS: stock_quantity = 3
        end
        CS->>+PA: insertPayment(order_id, payment_method = card, payment_status = pending, payment_amount = 2400.00)
        PA-->>-CS: payment_id
        CS-)TM: schedule(payment_id, ordered_at + 15 хв) (REQ-13)
        Note over CS,TM: у payment немає часу створення, тому відлік — від ordered_at, тієї самої транзакції. Якщо commit не вдасться, таймер не знайде payment і нічого не змінить (adr/0002)
        CS->>CS: commit()
    end
    CS-->>-UI: OrderCreated(order_id, payment_id, payment_amount = 2400.00)
    UI->>UI: clearCart() (REQ-17)
    UI-->>-C: перенаправлення на сторінку шлюзу з payment_id і 2400.00 (REQ-10)
    C->>+GW: вводить дані картки на сторінці шлюзу
    GW-->>-C: сторінка результату оплати
    Note over C,GW: що бачить Клієнт після шлюзу — UC5, поза lab2. Порядок цієї відповіді й повідомлення магазину lab2 не задає
    GW->>+PS: notifyPayment(payment_id, результат = succeeded, signature)
    PS->>PS: T = now() — час отримання повідомлення
    PS->>PS: verifySignature(body, signature) — HMAC-SHA256, дійсний (REQ-21)
    critical по одному для payment (REQ-16)
        PS->>+PA: findByIdForUpdate(payment_id)
        PA-->>-PS: payment_status = pending, order_id
        PS->>+PA: updatePayment(payment_id, payment_status = succeeded, paid_at = T)
        PA-->>-PS: ok
        PS->>+OR: updateOrderStatus(order_id, order_status = paid) (REQ-11)
        OR-->>-PS: ok
        PS->>PS: commit()
    end
    PS-->>-GW: Accepted
```