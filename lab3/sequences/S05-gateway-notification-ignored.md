# S05. Повідомлення шлюзу нічого не змінює

Сценарій: `@S5` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги сценарію: REQ-15, REQ-16, REQ-21 · рендер: [`svg/S05.svg`](svg/S05.svg)

```mermaid
sequenceDiagram
    autonumber
    actor GW as Платіжний шлюз
    participant PS as PaymentService
    participant PA as payment

    Note over GW,PA: Дано: стан order, payment і paid_at із рядка «Приклади» S5
    GW->>+PS: notifyPayment(payment_id, результат, signature)
    PS->>PS: verifySignature(body, signature) — HMAC-SHA256
    alt рядок 4 — підпис недійсний, payment pending (REQ-21)
        PS->>PS: result = Rejected(підпис недійсний)
        Note over PS,PA: payment і order не читаються й не змінюються
    else рядки 1–3 — підпис дійсний
        critical по одному для payment (REQ-16)
            PS->>+PA: findByIdForUpdate(payment_id)
            PA-->>-PS: payment_status, paid_at
            alt рядок 1 — failed, order cancelled, шлюз «оплату підтверджено»
                Note over PS: payment уже не pending — нічого не змінюється (REQ-15)
            else рядок 2 — succeeded, order paid, шлюз «банк відхилив»
                Note over PS: payment уже не pending — нічого не змінюється (REQ-15)
            else рядок 3 — succeeded, order paid, шлюз «оплату підтверджено»
                Note over PS: повтор того самого результату — нічого не змінюється (REQ-15)
            end
            PS->>PS: commit()
        end
        PS->>PS: result = Accepted
    end
    PS-->>-GW: result
```