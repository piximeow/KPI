# S02. Клієнт оплачує замовлення карткою

Сценарій: `@S2` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги: REQ-03, REQ-04, REQ-10, REQ-11, REQ-13, REQ-16, REQ-21 · рендер: [`svg/S02.svg`](svg/S02.svg)

```mermaid
sequenceDiagram
    autonumber
    actor C as Клієнт
    participant UI as CheckoutUI
    participant CS as CheckoutService
    participant TM as PaymentTimer
    actor GW as Платіжний шлюз
    participant PS as PaymentService
    participant PA as payment
    participant OR as order

    Note over C,OR: Дано: у Клієнта address у Києві, у кошику 2 шт. «Навушники» (price 1200.00)
    C->>UI: обирає address «Київ» і card, натискає «Підтвердити»
    UI->>+CS: confirmOrder(customerId, addressId, card, кошик)
    Note over CS: перевірки й транзакція створення — як у S01: order new, order_item, payment card pending 2400.00
    CS-)TM: schedule(paymentId, now + 15 хв) (REQ-13)
    CS-->>-UI: OrderCreated(orderId, paymentId, card, 2400.00)
    UI->>UI: clearCart()
    UI-)GW: redirect(paymentId, 2400.00) (REQ-10)
    C->>GW: вводить дані картки на сторінці шлюзу
    GW->>+PS: notifyPayment(paymentId, succeeded, signature)
    PS->>PS: verifySignature(body, signature) — HMAC-SHA256 (REQ-21)
    critical по одному для payment (REQ-16)
        PS->>+PA: findByIdForUpdate(paymentId)
        PA-->>-PS: pending, orderId
        alt payment_status = pending
            PS->>+PA: updatePayment(paymentId, succeeded, paid_at = now)
            PA-->>-PS: ok
            PS->>+OR: updateOrderStatus(orderId, paid)
            OR-->>-PS: ok
        else вже не pending
            Note over PS: нічого не змінюється — див. S05 (REQ-15)
        end
    end
    PS-->>-GW: 200 OK
```