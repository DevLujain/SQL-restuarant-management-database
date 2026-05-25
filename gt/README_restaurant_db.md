# Restaurant Management System — Oracle SQL / PL/SQL

A fully normalised relational database system for restaurant operations, built as part of a Database Systems course project. Covers the full backend data layer: schema design, automated stock management, transactional order processing, and reporting views.

---

## Features

### Schema Design
- **14 tables** covering the full restaurant domain: customers, staff, roles, menu items, ingredients, orders, payments, purchase orders, reservations, and a help/metadata table
- Enforced referential integrity via **foreign key constraints** with cascaded deletes where appropriate
- Domain validation via **CHECK constraints** (e.g. non-negative stock levels, valid payment statuses, valid order types)
- **Unique constraints** on business keys (supplier names, role names, payment-to-order mapping)

### Automated ID Generation
- **10 sequences** (`SEQ_SUPPLIER`, `SEQ_CUSTOMER`, `SEQ_STAFF`, etc.) providing auto-incrementing primary keys
- **10 BEFORE INSERT triggers** implementing the sequence pattern cleanly on each entity table

### Business Logic — Triggers
| Trigger | Event | Purpose |
|---|---|---|
| `TRG_ORDERLINE_DEDUCT_STOCK` | AFTER INSERT on ORDER_LINE | Deducts ingredient stock proportionally when an order line is placed; raises error on insufficient stock |
| `TRG_PURCHASE_ADD_STOCK` | AFTER INSERT on PURCHASEDETAIL | Restores ingredient stock when a purchase order detail is received |

### Stored Procedures
| Procedure | Purpose |
|---|---|
| `SP_PLACE_ORDER` | Creates a payment record and a linked order in a single transaction; returns the new order ID |
| `SP_RECEIVE_PO` | Creates a purchase order record linked to a supplier and staff member; returns the new PO ID |

### Function
| Function | Purpose |
|---|---|
| `FN_ORDER_TOTAL(p_orderid)` | Returns the computed total for a given order (SUM of quantity × unit price across all order lines) |

### Reporting Views
| View | Description |
|---|---|
| `VW_MENU` | Clean menu listing (ID, name, description, price) |
| `VW_STOCK_STATUS` | Ingredient stock levels with `OK` / `REORDER` status flag based on reorder threshold |
| `VW_ORDER_SUMMARY` | Joined order view including staff name, payment method, status, and amount |

---

## Schema Overview

```
SUPPLIER ──< INGREDIENT ──< INGREDIENT_ITEM >── MENUITEM
                                                    │
STAFF ──< STAFF_ROLE >── ROLE          ORDER_LINE >─┘
  │                                        │
  └──< ORDERS >── PAYMENT            ORDERS ──> STAFF
                                           └──> PAYMENT

CUSTOMER ──< RESERVATION
PURCHASEORDER ──< PURCHASEDETAIL ──> INGREDIENT
```

---

## Technologies

- **Oracle Database** (tested on Oracle 19c / Oracle XE)
- **SQL** — DDL (CREATE TABLE, VIEW, SEQUENCE), DML (INSERT, UPDATE), DCL
- **PL/SQL** — triggers, stored procedures, anonymous blocks, exception handling

---

## How to Run

1. Open Oracle SQL Developer or SQL*Plus
2. Connect to your schema
3. Run the full script:
   ```sql
   @restaurant_db.sql
   ```
   The script is **idempotent** — it safely drops all objects before recreating them. Safe to re-run.

4. Verify with the included queries:
   ```sql
   SELECT * FROM VW_MENU ORDER BY MENUITEMID;
   SELECT * FROM VW_STOCK_STATUS ORDER BY STATUS, INGREDIENTID;
   SELECT * FROM VW_ORDER_SUMMARY ORDER BY ORDERID DESC;
   ```

---

## Sample Data Included

- 3 suppliers, 3 staff members, 4 roles, 2 customers
- 3 menu items (Shrooms Asada, Spicy Asian Taco, Protein Packed)
- 8 ingredients with stock levels and reorder thresholds
- Ingredient-to-menu-item mappings with per-item quantities
- 1 reservation, 1 completed dine-in order (with automatic stock deduction via trigger), 1 purchase order receipt (with automatic stock restoration via trigger)

---

## Key Design Decisions

- **Trigger-based stock management** keeps inventory consistent without requiring application-level logic — any order line insertion automatically deducts the correct ingredient quantities based on the `INGREDIENT_ITEM` recipe table
- **Payment created before order** in `SP_PLACE_ORDER` to satisfy the FK constraint; amount is updated post-insertion once order lines are known
- **`SET DEFINE OFF`** at the top prevents Oracle from interpreting `&` in contact strings as substitution variables
- **Idempotent teardown** uses `EXCEPTION WHEN OTHERS THEN NULL` to gracefully skip missing objects on first run

---

## Author

Lujain Alhumaidah — Database Systems coursework, University of Malaya
