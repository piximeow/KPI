# S04. Оформлення відхилено

Сценарій: `@S4` у [`lab2/features/uc2-place-order.feature`](../../lab2/features/uc2-place-order.feature) · вимоги: REQ-02, REQ-06, REQ-07, REQ-08, REQ-09, REQ-19, REQ-20, REQ-22 · рендер: [`svg/S04.svg`](svg/S04.svg)

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

    Note over C,PA: Дано: стан із рядка «Приклади» S4. Перевірки в порядку REQ-22, показується перша, що виконалась
    alt «Оформити», у customer немає address (REQ-02)
        C->>UI: натискає «Оформити»
        UI->>+CS: openCheckout(customerId)
        CS->>+AD: findByCustomer(customerId)
        AD-->>-CS: []
        CS-->>-UI: NoAddress
        UI-->>C: «Додайте адресу доставки в профілі»
    else «Підтвердити»
        C->>UI: натискає «Підтвердити»
        UI->>+CS: confirmOrder(customerId, addressId?, payment_method?, кошик)
        CS->>+OR: countOrderedSince(ordered_at ≥ now − 60 с)
        OR-->>-CS: кількість order
        alt вже 100 order за 60 с (REQ-20)
            CS-->>UI: Refused(«Забагато замовлень, спробуйте за хвилину»)
        else не обрано address або спосіб оплати (REQ-19)
            CS-->>UI: Refused(«Оберіть адресу й спосіб оплати»)
        else кошик порожній (REQ-08)
            CS-->>UI: Refused(«Кошик порожній»)
        else перевірки форми пройдено
            critical транзакція REQ-03–REQ-05 (REQ-06)
                CS->>+PR: lockForUpdate(productIds)
                PR-->>-CS: price, stock_quantity кожного product
                alt quantity > stock_quantity хоча б для одного (REQ-07)
                    CS-->>UI: Refused(«product_name: доступно N шт.» для кожного)
                else сума quantity × price = 0.00 (REQ-09)
                    CS-->>UI: Refused(«Замовлення на 0.00 грн не оформлюється»)
                else збій під час створення (REQ-06)
                    CS->>+OR: insertOrder(new, ordered_at, addressId)
                    OR-->>-CS: orderId
                    CS->>+OI: insertOrderItem(orderId, productId, quantity, unit_price)
                    OI-->>-CS: orderItemId
                    CS->>+PA: insertPayment(orderId, cash_on_delivery, pending, сума)
                    PA-->>-CS: помилка БД
                    CS->>CS: rollback() — нічого не зберігається
                    CS-->>UI: Refused(«Не вдалося оформити замовлення, спробуйте ще раз»)
                end
            end
        end
        deactivate CS
        UI-->>C: показує повідомлення, кошик не змінює
    end
```