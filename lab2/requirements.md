# Вимоги до UC2 «Оформити замовлення»

## EARS

| ID | Вимога | Шаблон | Сутності lab1 |
|----|--------|--------|---------------|
| REQ-01 | WHEN Клієнт підтверджує замовлення, система SHALL створити `order` зі статусом `new`, `ordered_at` = поточний час і `address_id` обраної адреси. | WHEN | order, address |
| REQ-02 | WHEN система створює `order`, система SHALL створити по одному `order_item` на кожен товар кошика з `quantity` з кошика і `unit_price` = поточна `product.price`. | WHEN | order_item, product |
| REQ-03 | WHEN система створює `order_item`, система SHALL зменшити `stock_quantity` його `product` на `quantity`. | WHEN | product, order_item |
| REQ-04 | IF `quantity` товару в кошику більша за його `stock_quantity`, THEN система SHALL не створювати `order` і показати Клієнту `product_name` і доступну кількість. | IF-THEN | product |
| REQ-05 | WHEN система створює `order`, система SHALL створити `payment` зі статусом `pending`, обраним `payment_method` і `payment_amount` = сума `quantity` × `unit_price` усіх його `order_item`. | WHEN | payment, order_item |
| REQ-06 | WHEN створено `payment` з `payment_method` = `card`, система SHALL перенаправити Клієнта на сторінку оплати Платіжного шлюзу з `payment_id` і `payment_amount`. | WHEN | payment |
| REQ-07 | WHEN Платіжний шлюз повідомляє про успішну оплату `payment`, система SHALL встановити `payment_status` = `succeeded`, `paid_at` = час повідомлення і `order_status` = `paid`. | WHEN | payment, order |
| REQ-08 | WHEN Платіжний шлюз повідомляє про відмову в оплаті `payment`, система SHALL встановити `payment_status` = `failed` і `order_status` = `cancelled`. | WHEN | payment, order |
| REQ-09 | IF Платіжний шлюз не повідомив результат через 15 хв після створення `payment` з `card`, THEN система SHALL встановити `payment_status` = `failed` і `order_status` = `cancelled`. | IF-THEN | payment, order |
| REQ-10 | WHEN система встановлює `order_status` = `cancelled`, система SHALL збільшити `stock_quantity` кожного товару на `quantity` його `order_item`. | WHEN | order, order_item, product |
| REQ-11 | WHEN Клієнт натискає «Оформити замовлення», система SHALL показати список `address` цього `customer` для вибору. | WHEN | customer, address |

Зв'язки на діаграмі: `include` UC2 → UC3 тримається на REQ-11; `extend` UC4 → UC2 [обрано `card`] — на REQ-06–REQ-09.

## NFR

| ID | Тема | Вимога |
|----|------|--------|
| NFR-01 | Пристрої й браузери | Оформлення працює в останніх двох версіях Chrome, Firefox, Safari, Edge на екранах шириною від 360 до 1920 px. |
| NFR-02 | Мова | Увесь текст оформлення — українською; суми в гривнях з двома знаками після коми (`2400.00 грн`). |
| NFR-03 | Час відгуку | p95 від підтвердження до показу номера `order` ≤ 2 с при 50 одночасних оформленнях, кошик до 20 позицій, вимір на сервері протягом 10 хв. |
| NFR-04 | Навантаження | Система обробляє 100 оформлень на хвилину; понад цю межу нові запити отримують повідомлення «Спробуйте за хвилину», уже розпочаті — завершуються. |
| NFR-05 | Безпека | Дані картки не зберігаються в системі — їх вводять лише на сторінці шлюзу; повідомлення шлюзу без дійсного підпису відхиляються; обмін — лише HTTPS. |