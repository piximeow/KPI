# S03. Оплата карткою завершується подією шлюзу або таймера

Сценарій: `@S3` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги: REQ-11, REQ-12, REQ-13, REQ-14, REQ-16 · рендер: [`svg/S03.svg`](svg/S03.svg)

```mermaid
sequenceDiagram
    autonumber
    actor GW as Платіжний шлюз
    participant TM as PaymentTimer
    participant PS as PaymentService
    participant PA as payment
    participant OR as order
    participant OI as order_item
    participant PR as product

    Note over GW,PR: Дано: order new, payment card pending 2400.00 створено о 12:00:00, stock_quantity «Навушники» 3
    alt 12:05:00 — шлюз повідомляє про відмову банку
        GW->>+PS: notifyPayment(paymentId, failed, signature)
        PS->>PS: verifySignature(body, signature)
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(paymentId)
            PA-->>-PS: pending, orderId
            PS->>+PA: updatePayment(paymentId, failed)
            PA-->>-PS: ok
            PS->>+OR: updateOrderStatus(orderId, cancelled) (REQ-12)
            OR-->>-PS: ok
            PS->>+OI: findByOrder(orderId)
            OI-->>-PS: [«Навушники», quantity 2]
            loop кожен order_item (REQ-14)
                PS->>+PR: increaseStock(productId, 2)
                PR-->>-PS: stock_quantity 5
            end
        end
        PS-->>-GW: 200 OK
    else 12:14:59 — подій ще немає
        Note over TM,PR: таймер ще не спрацював: payment pending, order new, stock_quantity 3
    else 12:15:00 — таймер, шлюз мовчить
        TM->>+PS: onPaymentTimeout(paymentId)
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(paymentId)
            PA-->>-PS: pending, orderId
            PS->>+PA: updatePayment(paymentId, failed)
            PA-->>-PS: ok
            PS->>+OR: updateOrderStatus(orderId, cancelled) (REQ-13)
            OR-->>-PS: ok
            PS->>+OI: findByOrder(orderId)
            OI-->>-PS: [«Навушники», quantity 2]
            loop кожен order_item (REQ-14)
                PS->>+PR: increaseStock(productId, 2)
                PR-->>-PS: stock_quantity 5
            end
        end
        PS-->>-TM: done (до 12:16:00)
    else 12:15:00 — шлюз повідомляє успіх, слідом спрацьовує таймер
        GW->>+PS: notifyPayment(paymentId, succeeded, signature)
        PS->>PS: verifySignature(body, signature)
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(paymentId)
            PA-->>-PS: pending, orderId
            PS->>+PA: updatePayment(paymentId, succeeded, paid_at = now)
            PA-->>-PS: ok
            PS->>+OR: updateOrderStatus(orderId, paid) (REQ-11)
            OR-->>-PS: ok
        end
        PS-->>-GW: 200 OK
        TM->>+PS: onPaymentTimeout(paymentId)
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(paymentId)
            PA-->>-PS: succeeded — не pending, нічого не змінюється
        end
        PS-->>-TM: done
    end
```