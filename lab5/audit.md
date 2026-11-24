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
| шлюз → `IPaymentNotify` → `PaymentWebhook` → `IPaymentEvents` | `notifyPayment` | S02, S03, S05 |
| `CheckoutService` → `IAddresses` | `findByCustomer` | S01, S02, S04 |
| `CheckoutService` → `IProducts` | `lockForUpdate`, `decreaseStock` | S01, S02, S04 |
| `CheckoutService` → `IOrders` | `countOrderedSince`, `insertOrder`, `insertOrderItem` | S01, S02, S04 |
| `CheckoutService` → `IPayments` | `insertPayment` | S01, S02, S04 |
| `CheckoutService` → `ITimer` | `schedule` | S02 |
| `PaymentTimer` → `IPaymentEvents` | `onPaymentTimeout` | S03 |
| `PaymentService` → `IPayments` | `findByIdForUpdate`, `updatePayment` | S02, S03, S05 |
| `PaymentService` → `IOrders` | `updateOrderStatus`, `findByOrder` | S02, S03 |
| `PaymentService` → `IProducts` | `increaseStock` | S03 |
| репозиторії → `SQL` | доступ до таблиць | — (БД у lab3 не учасник) |

Кожна операція — у того самого учасника, що в lab3. Внутрішні кроки (`checkForm`, `verifySignature`, `clearCart`, `commit`, `rollback` …) — не інтерфейси.

## 3. Межа, інтерфейси, шари
- **Межа:** Клієнт і шлюз — поза рамкою, `CheckoutUI` (браузерна частина магазину) — всередині. Межу перетинають лише три стрілки до інтерфейсів: Клієнт → `IPages`, `CheckoutUI` → `IPaymentPage`, шлюз → `IPaymentNotify`, усе по HTTPS. `ICheckout` — усередині, між браузером і сервером. Дані картки йдуть лише Клієнт → `IPaymentPage`, повз межу; `IPaymentEvents` з `onPaymentTimeout` ззовні недосяжний.
- **Provided:** кожен компонент має один «льодяник» з операціями; **required:** usage-стрілка `..>` до чужого інтерфейсу.
- **Шари:** UI → API → сервіси → Data Access → БД згори вниз; між шарами всередині межі стрілки йдуть лише вниз, за межу вгору — лише `CheckoutUI` → `IPaymentPage`; усередині шару сервісів — `CheckoutService → PaymentTimer → PaymentService`, назад ніхто не кличе, циклів немає.

## 4. Залишкові розбіжності
- **`CheckoutAPI`, `PaymentWebhook`** — нові: у lab3 відповідника немає, у lab4 вони не виділені з доріжки «Магазин» (`adr/0001`).
- **`OrderRepository`** відповідає двом учасникам lab3 (`adr/0002`).
- **`IPaymentEvents`** ширший за потреби кожного клієнта: `PaymentWebhook` кличе лише `notifyPayment`, `PaymentTimer` — лише `onPaymentTimeout`; у lab3 обидві приймає один учасник.
- **`SQL`** без операцій lab3 — БД у lab3 не моделювалась.
- **Таймер після перезапуску:** повторна постановка для `pending` — поза моделлю, припущення з `lab3/adr/0002`.

## Топ-3 розбіжності, виправлені через spec (було → стало)
1. **Операції не з lab3** ↔ S01–S05: було — `createOrder`, `handleNotification`, репозиторії без операцій; стало — операції lab3 у кожному інтерфейсі.
2. **Шлях даних картки** ↔ NFR-05, S02: було — `UI ..> GW` без інтерфейсу; стало — `IPaymentPage` шлюзу і стрілка Клієнт → `IPaymentPage`.
3. **`customer` у Data Access** ↔ lab1, lab3: було — `CustomerRepository (customer, address)`, хоча до `customer` у lab3 звернень немає; стало — `AddressRepository`, а `customer` лишився в `ShopDB` як ціль FK.
Коміти кожної — `ai/prompts.md`, «Топ-3 розбіжності».