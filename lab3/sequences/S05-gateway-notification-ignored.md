# S05. Повідомлення шлюзу нічого не змінює

Сценарій: `@S5` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги: REQ-15, REQ-16, REQ-21 · рендер: [`svg/S05.svg`](svg/S05.svg)

```mermaid
sequenceDiagram
    autonumber
    actor GW as Платіжний шлюз
    participant PS as PaymentService
    participant PA as payment

    Note over GW,PA: Дано: стан order, payment і paid_at із рядка «Приклади» S5
    GW->>+PS: notifyPayment(paymentId, результат, signature)
    PS->>PS: verifySignature(body, signature) — HMAC-SHA256
    alt підпис недійсний (REQ-21)
        PS-->>GW: 401 — payment і order не змінено
    else підпис дійсний
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(paymentId)
            PA-->>-PS: payment_status, paid_at
            alt payment_status = pending
                Note over PS: обробка як у S02 або S03
            else failed або succeeded (REQ-15)
                Note over PS: payment, order і stock_quantity не змінюються
            end
        end
        PS-->>GW: 200 OK
    end
    deactivate PS
```