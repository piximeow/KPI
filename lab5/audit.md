# Аудит компонентної моделі

Відповіді чату й рішення за кожною знахідкою — `ai/prompts.md` (П1, П2). Кожне виправлення спершу змінювало spec, потім діаграму.

## 1. Компоненти → lab1–lab4
| Компонент | lab1 (дані) | lab2 (вимоги) | lab3 (учасник) | lab4 (доріжка) |
|---|---|---|---|---|
| `CheckoutUI` | — | REQ-10, 17, 18 | `CheckoutUI` (S01, S02, S04) | «Клієнт (CheckoutUI)» |
| `CheckoutAPI` | — | NFR-05 (HTTPS) | **нового**: UI кличе `CheckoutService` напряму (`adr/0001`) | окремо немає, частина «Магазину» |
| `PaymentWebhook` | — | REQ-11, 12, 15, 21 (вхід), NFR-05 | **нового**: шлюз кличе `PaymentService` напряму (`adr/0001`) | окремо немає, частина «Магазину» |
| `CheckoutService` | `address`, `product`, `order`, `order_item`, `payment` | REQ-01–09, 13, 19, 20, 22 | `CheckoutService` (S01, S02, S04) | «Магазин» |
| `PaymentTimer` | `order.ordered_at` | REQ-13 | `PaymentTimer` (S02, S03) | подія `onPaymentTimeout` |
| `PaymentService` | `payment`, `order`, `order_item`, `product` | REQ-11–16, 21 | `PaymentService` (S02, S03, S05) | «Магазин» |
| `AddressRepository` | `address` | REQ-01, 02, 19 | учасник `address` | — |
| `ProductRepository` | `product` | REQ-05, 07, 14 | учасник `product` | — |
| `OrderRepository` | `order`, `order_item` | REQ-03, 11–14, 20 | учасники `order` і `order_item` — в одному (`adr/0002`) | — |
| `PaymentRepository` | `payment` | REQ-04, 11–13, 15, 16 | учасник `payment` | — |
| `ShopDB` | `customer`, `address`, `product`, `order`, `order_item`, `payment` | — | — (сховище учасників-сутностей) | — |
| Платіжний шлюз | — | REQ-10, 11, 12, 21, NFR-05 | Платіжний шлюз (S02, S03, S05) | «Платіжний шлюз» |

`category` і `review` з lab1 в UC2 не беруть участі, тож їх у `ShopDB` не показано.

## 2. Залежності → lab3
| Залежність | Операції | Де в lab3 |
|---|---|---|
| Клієнт → `IPages` | «Оформити», «Підтвердити» | S01, S02, S04 |
| Клієнт → `IPaymentPage` | дані картки | S02 («вводить дані картки на сторінці шлюзу») |
| `CheckoutUI` → `ICheckout` → `CheckoutAPI` → `ICheckoutService` | `openCheckout`, `confirmOrder` | S01, S02, S04 |
| `CheckoutUI` → `IPaymentPage` | перехід з `payment_id`, `payment_amount` | S02 — перенаправлення Клієнта (REQ-10); lab4 — «перехід на сторінку оплати» |
| шлюз → `IPaymentNotify` → `PaymentWebhook` → `IPaymentNotifications` | `notifyPayment` | S02, S03, S05 |
| `CheckoutService` → `IAddresses` | `findByCustomer` | S01, S02, S04 |
| `CheckoutService` → `IProducts` | `lockForUpdate`, `decreaseStock` | S01, S02, S04 |
| `CheckoutService` → `IOrders` | `countOrderedSince`, `insertOrder`, `insertOrderItem` | S01, S02, S04 |
| `CheckoutService` → `IPayments` | `insertPayment` | S01, S02, S04 |
| `CheckoutService` → `ITimer` | `schedule` | S02 |
| `PaymentTimer` → `IPaymentTimeout` | `onPaymentTimeout` | S03 |
| `PaymentService` → `IPayments` | `findByIdForUpdate`, `updatePayment` | S02, S03, S05 |
| `PaymentService` → `IOrders` | `updateOrderStatus`, `findByOrder` | S02, S03 |
| `PaymentService` → `IProducts` | `increaseStock` | S03 |
| репозиторії → `SQL` | доступ до таблиць | — (БД у lab3 не учасник) |

Кожна операція — у того самого учасника, що в lab3. Внутрішні кроки (`checkForm`, `verifySignature`, `clearCart`, `commit`, `rollback` …) — не інтерфейси.

## 3. Межа, інтерфейси, шари
- **Межа:** Клієнт і шлюз — поза рамкою, `CheckoutUI` (браузерна частина магазину) — всередині. Межу перетинають лише три стрілки до інтерфейсів: Клієнт → `IPages`, `CheckoutUI` → `IPaymentPage`, шлюз → `IPaymentNotify`, усе по HTTPS. `ICheckout` — усередині, між браузером і сервером. Дані картки йдуть лише Клієнт → `IPaymentPage`, повз межу; `onPaymentTimeout` ззовні недосяжний — `IPaymentTimeout` використовує лише `PaymentTimer`.
- **Provided:** кожен компонент має «льодяник», назва й операції — у підписі біля нього, `PaymentService` — два: для шлюзу й для таймера; **required:** usage-стрілка `..>` до чужого інтерфейсу.
- **Шари:** UI → API → сервіси → Data Access → БД згори вниз; між шарами всередині межі стрілки йдуть лише вниз, за межу вгору — лише `CheckoutUI` → `IPaymentPage`; усередині шару сервісів — `CheckoutService → PaymentTimer → PaymentService`, назад ніхто не кличе, циклів немає.

## 4. Нефункціональні вимоги
| NFR | Де в моделі |
|---|---|
| NFR-01 браузери, NFR-02 мова | `CheckoutUI` (сторінки UC2); сторінка шлюзу — поза магазином |
| NFR-03 p95 ≤ 2 с, NFR-04 500 Клієнтів і 100 підтверджень/хв | структуру не змінюють; вузьке місце — `lockForUpdate` по `product` в одній `ShopDB`; межу 100 за 60 с перевіряє `countOrderedSince` (REQ-20) |
| NFR-05 безпека | HTTPS на всіх стрілках через межу, дані картки лише Клієнт → `IPaymentPage`, підпис HMAC-SHA256 на `IPaymentNotify`, перевірка — у `PaymentService` (REQ-21) |

## 5. Залишкові розбіжності
- **`CheckoutAPI`, `PaymentWebhook`** — нові: у lab3 відповідника немає, у lab4 вони не виділені з доріжки «Магазин» (`adr/0001`). `ICheckoutService` повторює операції `ICheckout`, бо `CheckoutAPI` лише передає виклик.
- **`OrderRepository`** відповідає двом учасникам lab3 (`adr/0002`).
- **`SQL`** без операцій lab3 — БД у lab3 не моделювалась.
- **Вхід Клієнта** ↔ lab1 (немає облікових даних), lab2 (вхід поза моделлю): `openCheckout(customer_id)` і `confirmOrder(customer_id, …)` приходять з браузера, і `CheckoutAPI` довіряє `customer_id`. Наслідок: хто знає чужий `customer_id`, оформить замовлення на чужу адресу. Закриває це окремий прецедент входу з компонентом ідентифікації — поза UC2.
- **Таймер після перезапуску** ↔ REQ-13, `lab3/adr/0002`: таймер живе поза БД. Наслідок: після перезапуску `pending` не скасовуються — REQ-13 («не пізніше ніж через 1 хв») і REQ-14 не виконуються, доки таймери не поставлено знову; повторна постановка — припущення, поза моделлю.

## Топ-3 розбіжності між моделями (як узгоджено)
1. **Нові `CheckoutAPI` і `PaymentWebhook`** ↔ lab3 (UI кличе `CheckoutService`, шлюз — `PaymentService` напряму), lab4 (окремо немає): названо новими в spec, обґрунтовано в `adr/0001`, позначено в таблиці 1.
2. **Дві події `PaymentService`** ↔ lab3 (шлюз і таймер кличуть одного учасника): було — один `IPaymentEvents`, і `PaymentWebhook` бачив `onPaymentTimeout`; стало — `IPaymentNotifications` і `IPaymentTimeout`, недосяжність таймауту видно з діаграми.
3. **Вхід Клієнта** ↔ lab1, lab2, lab3 (`customer_id` від браузера): компонент ідентифікації не додано — у lab1 і lab2 для нього немає ні даних, ні вимог; розбіжність і наслідок записано (розділ 5, spec — «Поза моделлю»).
Коміти кожної — `ai/prompts.md`, «Топ-3 розбіжності».