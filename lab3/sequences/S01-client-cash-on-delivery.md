# S01. Клієнт оформлює замовлення з оплатою при отриманні

Сценарій: `@S1` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги: REQ-01, REQ-03, REQ-04, REQ-05, REQ-17, REQ-18, REQ-22 · рендер: [`svg/S01.svg`](svg/S01.svg)

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

    Note over C,PA: Дано: customer «Олена Коваль» з address у Києві, у кошику 2 шт. «Навушники» (price 1200.00, stock_quantity 5)
    C->>UI: натискає «Оформити»
    UI->>+CS: openCheckout(customerId)
    CS->>+AD: findByCustomer(customerId)
    AD-->>-CS: [address «Київ»]
    CS-->>-UI: форма: address «Київ», способи card і cash_on_delivery (REQ-01)
    UI-->>C: показує форму
    Note over PR: price «Навушники» змінюється на 1300.00
    C->>UI: обирає address «Київ» і cash_on_delivery, натискає «Підтвердити»
    UI->>+CS: confirmOrder(customerId, addressId, cash_on_delivery, кошик)
    CS->>+OR: countOrderedSince(ordered_at ≥ now − 60 с)
    OR-->>-CS: < 100 (REQ-20 не виконується)
    CS->>CS: checkForm(addressId, payment_method, кошик) — REQ-19, REQ-08 не виконуються
    critical транзакція REQ-03–REQ-05 (REQ-06)
        CS->>+PR: lockForUpdate(productIds)
        PR-->>-CS: «Навушники»: price 1300.00, stock_quantity 5
        CS->>CS: checkStockAndTotal() — REQ-07, REQ-09 не виконуються
        CS->>+OR: insertOrder(new, ordered_at = now, addressId)
        OR-->>-CS: orderId
        loop кожен product кошика
            CS->>+OI: insertOrderItem(orderId, productId, quantity 2, unit_price 1300.00)
            OI-->>-CS: orderItemId
            CS->>+PR: decreaseStock(productId, 2)
            PR-->>-CS: stock_quantity 3
        end
        CS->>+PA: insertPayment(orderId, cash_on_delivery, pending, 2600.00)
        PA-->>-CS: paymentId
    end
    CS-->>-UI: OrderCreated(orderId, cash_on_delivery)
    UI->>UI: clearCart() (REQ-17)
    alt payment_method = cash_on_delivery
        UI-->>C: показує номер замовлення orderId (REQ-18)
    else payment_method = card
        UI-->>C: перенаправлення на шлюз — див. S02
    end
```