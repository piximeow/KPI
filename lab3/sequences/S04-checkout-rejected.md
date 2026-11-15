# S04. Оформлення відхилено

Сценарій: `@S4` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги сценарію: REQ-02, REQ-06, REQ-07, REQ-08, REQ-09, REQ-19, REQ-20, REQ-22 · також: REQ-03, REQ-05 · кроки інтерфейсу поза lab2: disableConfirm(), enableConfirm() · рендер: [`svg/S04.svg`](svg/S04.svg)

```mermaid
sequenceDiagram
    autonumber
    actor C as Клієнт
    participant UI as CheckoutUI
    participant CS as CheckoutService
    participant AD as address
    participant OR as order
    participant PR as product
    participant OI as order_item
    participant PA as payment

    Note over C,PA: Дано: стан із рядка «Приклади» S4 (номер рядка — у гілці)
    alt рядок 1 — «Оформити», у customer немає address (REQ-02)
        C->>+UI: натискає «Оформити»
        UI->>+CS: openCheckout(customer_id)
        CS->>+AD: findByCustomer(customer_id)
        AD-->>-CS: []
        CS-->>-UI: NoAddress
        UI-->>-C: «Додайте адресу доставки в профілі»
    else рядки 2–11 — «Підтвердити»
        C->>+UI: натискає «Підтвердити»
        UI->>UI: disableConfirm()
        UI->>+CS: confirmOrder(customer_id, address_id, payment_method, кошик)
        Note over CS: умови перевіряються в порядку REQ-22, береться перша, що виконалась
        CS->>+OR: countOrderedSince(ordered_at ≥ now − 60 с)
        OR-->>-CS: кількість order
        alt рядки 10, 11 — уже 100 order за 60 с (REQ-20)
            Note over CS: рядок 11: не обрано й форму, але REQ-20 перша
            CS->>CS: result = «Забагато замовлень, спробуйте за хвилину»
        else рядки 2, 3, 4 — не обрано address або спосіб оплати (REQ-19)
            Note over CS: рядок 4: ще й кошик порожній, але REQ-19 перша
            CS->>CS: result = «Оберіть адресу й спосіб оплати»
        else рядок 5 — кошик порожній (REQ-08)
            CS->>CS: result = «Кошик порожній»
        else рядки 6, 7, 8, 9 — перевірки форми пройдено
            critical транзакція REQ-03–REQ-05 (REQ-06)
                CS->>+PR: lockForUpdate(product_id кожного товару кошика)
                PR-->>-CS: price, stock_quantity кожного product
                alt рядки 6, 8 — quantity > stock_quantity (REQ-07)
                    Note over CS: рядок 8: сума ще й 0.00, але REQ-07 перша
                    CS->>CS: rollback()
                    CS->>CS: result = «product_name: доступно stock_quantity шт.» для кожного
                else рядок 7 — сума quantity × price = 0.00 (REQ-09)
                    CS->>CS: rollback()
                    CS->>CS: result = «Замовлення на 0.00 грн не оформлюється»
                else рядок 9 — перевірки пройдено, під час insertPayment збій БД
                    CS->>+OR: insertOrder(order_status = new, ordered_at = now, address_id)
                    OR-->>-CS: order_id
                    CS->>+OI: insertOrderItem(order_id, product_id, quantity = 2, unit_price = 1200.00)
                    OI-->>-CS: order_item_id
                    CS->>+PR: decreaseStock(product_id, quantity = 2)
                    PR-->>-CS: stock_quantity = 3
                    CS->>+PA: insertPayment(order_id, payment_method = cash_on_delivery, payment_status = pending, payment_amount = 2400.00)
                    PA-->>-CS: помилка БД
                    CS->>CS: rollback() — order, order_item і decreaseStock скасовано
                    CS->>CS: result = «Не вдалося оформити замовлення, спробуйте ще раз»
                end
            end
        end
        CS-->>-UI: Refused(result)
        UI->>UI: enableConfirm() — кошик не змінено
        UI-->>-C: показує повідомлення result
    end
```