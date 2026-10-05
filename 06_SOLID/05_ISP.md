# Interface Segregation Principle

Clients should not be forced to depend on methods they do not use. Fat, monolithic interfaces force components to mock or implement irrelevant methods, increasing coupling and test fragility.

## Application to SALESTORM Repositories

We apply ISP heavily to the repository/provider boundaries in the LLD design. We prefer focused, domain-specific interfaces.

### Better (Follows ISP):
- `InventoryRepository`
- `ReservationRepository`
- `PaymentRepository`
- `OrderRepository`

*(Note: Exact interface names are architectural design artifacts representing data access patterns in the LLD).*

### Bad (Violates ISP):
Instead of one giant interface:
```text
MegaCommerceRepository
├── saveInventory()
├── saveReservation()
├── savePayment()
├── saveOrder()
├── findShipment()
└── sendNotification()
```

## Why Focused Interfaces Help

- **Smaller contracts:** Easier to understand and implement.
- **Easier testing:** When testing the `InventoryService`, developers only mock the `InventoryRepository`. They don't have to stub out `sendNotification()`.
- **Lower coupling:** Changes to order persistence don't recompile or affect inventory logic.
- **Clear ownership:** Maps directly to our domain boundaries.
- **Easier replacement:** A specific repository can be swapped out (e.g., moving orders to a NoSQL store while keeping inventory in PostgreSQL) without touching a mega-interface.
- **Better service boundaries:** Payment abstractions are kept entirely separate from unrelated inventory/order concerns.
