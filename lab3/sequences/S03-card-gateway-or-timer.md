# S03. Оплата карткою завершується подією шлюзу або таймера

Сценарій: `@S3` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги сценарію: REQ-11, REQ-12, REQ-13, REQ-14, REQ-16 · також: REQ-21 · рендер: [`svg/S03.svg`](svg/S03.svg)

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

    Note over GW,PR: Дано: order new, payment card pending 2400.00 створено о 12:00:00, stock_quantity «Навушники» = 3
    alt рядок 1 — о 12:05:00 шлюз повідомляє про відмову банку
        GW->>+PS: notifyPayment(payment_id, результат = failed, signature)
        PS->>PS: verifySignature(body, signature) — дійсний
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(payment_id)
            PA-->>-PS: payment_status = pending, order_id
            PS->>+PA: updatePayment(payment_id, payment_status = failed)
            PA-->>-PS: ok
            PS->>+OR: updateOrderStatus(order_id, order_status = cancelled) (REQ-12)
            OR-->>-PS: ok
            PS->>+OI: findByOrder(order_id)
            OI-->>-PS: [product_id «Навушники», quantity = 2]
            loop кожен order_item (REQ-14)
                PS->>+PR: increaseStock(product_id, quantity = 2)
                PR-->>-PS: stock_quantity = 5
            end
            PS->>PS: commit()
        end
        PS-->>-GW: Accepted
    else рядок 2 — настає 12:14:59 без повідомлення шлюзу
        Note over TM,PR: таймер ще не спрацював: payment pending, order new, stock_quantity = 3
    else рядок 3 — настає 12:16:00 без повідомлення шлюзу
        Note over TM: таймер спрацьовує між 12:15:00 і 12:16:00 (REQ-13)
        TM-)+PS: onPaymentTimeout(payment_id)
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(payment_id)
            PA-->>-PS: payment_status = pending, order_id
            PS->>+PA: updatePayment(payment_id, payment_status = failed)
            PA-->>-PS: ok
            PS->>+OR: updateOrderStatus(order_id, order_status = cancelled) (REQ-13)
            OR-->>-PS: ok
            PS->>+OI: findByOrder(order_id)
            OI-->>-PS: [product_id «Навушники», quantity = 2]
            loop кожен order_item (REQ-14)
                PS->>+PR: increaseStock(product_id, quantity = 2)
                PR-->>-PS: stock_quantity = 5
            end
            PS->>PS: commit()
        end
        deactivate PS
        Note over TM,PR: о 12:16:00 payment failed, order cancelled, stock_quantity = 5
    else рядок 4 — о 12:15:00 шлюз повідомляє успіх, слідом спрацьовує таймер
        GW->>+PS: notifyPayment(payment_id, результат = succeeded, signature)
        PS->>PS: T = now() — час отримання повідомлення
        PS->>PS: verifySignature(body, signature) — дійсний
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
        TM-)+PS: onPaymentTimeout(payment_id)
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(payment_id)
            PA-->>-PS: payment_status = succeeded
            Note over PS: payment уже не pending — таймер нічого не змінює
            PS->>PS: commit()
        end
        deactivate PS
    end
```