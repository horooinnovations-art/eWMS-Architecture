# Data model

The main entities are shown below; supporting tables (settings, audit log, rate limits,
migrations) are left out.

```mermaid
erDiagram
    SUPPLIER ||--o{ PURCHASE : "supplies"
    PURCHASE ||--|{ PURCHASE_ITEM : "contains"
    ITEM ||--o{ PURCHASE_ITEM : "ordered as"
    PURCHASE ||--o{ INVENTORY_BATCH : "received into"
    ITEM ||--o{ INVENTORY_BATCH : "stocked as"
    CATEGORY ||--o{ ITEM : "groups"
    MANUFACTURER ||--o{ ITEM : "makes"

    CUSTOMER ||--o{ SALES_ORDER : "places"
    SALES_ORDER ||--|{ SALES_ORDER_ITEM : "contains"
    ITEM ||--o{ SALES_ORDER_ITEM : "sold as"
    SALES_ORDER_ITEM ||--o{ FULFILMENT_ITEM : "allocated from"
    INVENTORY_BATCH ||--o{ FULFILMENT_ITEM : "supplies"
    SALES_ORDER ||--o{ INVOICE : "billed by"
    INVOICE ||--|{ INVOICE_ITEM : "contains"
    INVOICE ||--o{ PAYMENT : "settled by"

    PRICE_LIST ||--o{ PRICE_LIST_ITEM : "defines"
    CUSTOMER }o--|| PRICE_LIST : "uses"

    ACCOUNT ||--o{ LEDGER_ENTRY : "posted to"
    BANK }o--|| ACCOUNT : "maps to"

    ROLE ||--o{ USER : "assigned"
    ROLE ||--o{ ROLE_PERMISSION : "grants"
```

## Notes

| Area | Detail |
|---|---|
| **Identity** | Users have one role. Roles map to permission slugs, so features can be turned on per role without code changes. |
| **Stock** | Quantity lives on each **inventory batch** (batch number, expiry, cost, location), not on the item. An item's stock is the sum of its batches. |
| **Traceability** | Fulfilment rows link each sold quantity to the exact batch it came from, with a timestamp. Stock cards and recalls are built from these. |
| **Documents** | Order and invoice numbers are unique in the database. |
| **Money** | Amounts are stored as fixed-precision decimals; totals are stored on documents and checked against their lines. |
| **History** | Items, suppliers and customers are deactivated rather than deleted when they have history, so old reports stay complete. |
| **Schema changes** | Applied through ordered SQL migration files kept in version control. |
