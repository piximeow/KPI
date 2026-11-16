# S01. Клієнт оформлює замовлення з оплатою при отриманні

Сценарій: `@S1` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги сценарію: REQ-01, REQ-03, REQ-04, REQ-05, REQ-17, REQ-18 · також: REQ-06, REQ-07, REQ-08, REQ-09, REQ-19, REQ-20 · кроки поза lab2: disableConfirm() · рендер: [`svg/S01.svg`](svg/S01.svg)

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

    Note over C,PA: Дано: Клієнт має одну address у Києві, інший customer — address у Львові, у кошику 2 шт. «Навушники» (price 1200.00, stock_quantity 5)
    C->>+UI: натискає «Оформити»
    UI->>+CS: openCheckout(customer_id)
    CS->>+AD: findByCustomer(customer_id)
    AD-->>-CS: [address «Київ»]
    Note over AD: лише адреси цього customer, «Львів» не повертається
    CS-->>-UI: форма: address «Київ», payment_method card або cash_on_delivery
    UI-->>-C: форма (REQ-01)
    Note over PR: тим часом поза UC2 магазин змінює price «Навушники» на 1300.00
    C->>+UI: обирає address «Київ» і cash_on_delivery, натискає «Підтвердити»
    UI->>UI: disableConfirm() — кошик у браузері, друге натискання неможливе
    UI->>+CS: confirmOrder(customer_id, address_id, payment_method = cash_on_delivery, кошик)
    CS->>+OR: countOrderedSince(ordered_at ≥ now − 60 с)
    OR-->>-CS: кількість order
    Note over CS: менше 100 — REQ-20 не виконується
    CS->>CS: checkForm() — REQ-19, REQ-08 не виконуються
    critical транзакція REQ-03–REQ-05 (REQ-06)
        CS->>+PR: lockForUpdate(product_id кожного товару кошика)
        PR-->>-CS: price = 1300.00, stock_quantity = 5
        CS->>CS: checkStockAndTotal() — REQ-07, REQ-09 не виконуються
        CS->>+OR: insertOrder(order_status = new, ordered_at = now, address_id)
        OR-->>-CS: order_id
        loop кожен product кошика
            CS->>+OI: insertOrderItem(order_id, product_id, quantity = 2, unit_price = 1300.00)
            OI-->>-CS: order_item_id
            CS->>+PR: decreaseStock(product_id, quantity = 2)
            PR-->>-CS: stock_quantity = 3
        end
        CS->>+PA: insertPayment(order_id, payment_method = cash_on_delivery, payment_status = pending, payment_amount = 2600.00, paid_at = порожній)
        PA-->>-CS: payment_id
        CS->>CS: commit()
    end
    CS-->>-UI: OrderCreated(order_id)
    UI->>UI: clearCart() (REQ-17)
    UI-->>-C: номер замовлення order_id (REQ-18)
```