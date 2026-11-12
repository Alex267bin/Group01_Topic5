
# Sequence Diagram - Order Processing

## 1. Overview



It includes:
- Successful order and payment.
- Payment failure.
- Payment timeout after 15 minutes.


```mermaid
sequenceDiagram
    autonumber

    actor Customer
    participant UI as OrderUI
    participant C as OrderController
    participant I as InventoryService
    participant P as PaymentService
    participant T as TimerService


        else Payment failed
            P-->>C: paymentFailed
            C->>I: releaseStock(itemId, qty)