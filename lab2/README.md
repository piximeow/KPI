# Лабораторна робота 2: вимоги до UC5 «Оформити замовлення»

Продовження lab1 (ER-модель «Онлайн-магазин»). Вглиб розписано один прецедент — оформлення замовлення; модель даних lab1 не змінюється, розбіжності з нею розв'язуються вимогами.

```
lab2/
├── README.md
├── spec.md                     намір, актори, кроки UC5, критерії прийняття
├── use-cases.md                актори й цілі, перелік UC, межі UC5, рендер діаграми
├── diagrams/
│   ├── uc-diagram.puml         UC-діаграма (PlantUML)
│   └── uc-diagram.svg          рендер
├── features/
│   └── uc5-place-order.feature сценарії S1–S24 (Gherkin, вони ж user stories)
├── requirements.md             константи, EARS, NFR, «Поза обсягом»
├── traceability.md             матриця вимога ↔ сценарій/UC
├── audit.md                    мій підсумок аудиту: знайдено / прийнято / відхилено
├── DEFENSE.md                  точка входу для рев'ю
├── adr/
│   ├── 0001-guest-checkout-via-guest-customer.md
│   ├── 0002-gateway-result-is-source-of-truth.md
│   └── 0003-card-payment-scenarios-inside-uc5.md
└── ai/prompts.md               слід промптів з відповідями другого чату, топ-3 розбіжності, аудит виходу AI
```

Рендер діаграми: `java -jar plantuml.jar -tsvg diagrams/uc-diagram.puml`.