# ER-діаграма «Онлайн-магазин»

Модель за `spec.md`. GitHub малює діаграму автоматично з цього файлу.

```mermaid
erDiagram
    customer ||--o{ address : has
    address ||--o{ order : receives
    category }o--o{ product : contains
    order ||--|{ order_item : includes
    product ||--o{ order_item : "listed in"
    order ||--o{ payment : paid_by
    customer ||--o{ review : writes
    product ||--o{ review : has

    customer {
        int customer_id PK
        string first_name
        string last_name
        string email UK
        string phone
        datetime registered_at
    }

    address {
        int address_id PK
        int customer_id FK
        string country
        string city
        string street_line
        string postal_code
    }

    category {
        int category_id PK
        string category_name UK
    }

    product {
        int product_id PK
        string product_name
        string sku UK
        string description
        decimal price
        int stock_quantity
    }

    order {
        int order_id PK
        int address_id FK
        string order_status
        datetime ordered_at
    }

    order_item {
        int order_item_id PK
        int order_id FK
        int product_id FK
        int quantity
        decimal unit_price
    }

    payment {
        int payment_id PK
        int order_id FK
        string payment_method
        string payment_status
        decimal payment_amount
        datetime paid_at
    }

    review {
        int review_id PK
        int customer_id FK
        int product_id FK
        int rating
        string review_text
        datetime reviewed_at
    }
```