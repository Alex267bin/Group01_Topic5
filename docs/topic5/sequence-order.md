# Sequence Diagram - Order Processing

## 1. Overview

This sequence diagram describes the interaction between system components during the order lifecycle in the Flash Sale E-commerce system.

It includes:
- Successful order and payment.
- Payment failure.
- Payment timeout after 15 minutes.
- Stock unavailable.
- Automatic stock release.

## 2. Participants

| Participant | Responsibility |
|---|---|
| Customer | Initiates an order |
| OrderUI | Receives customer requests |
| OrderController | Controls order processing |
| InventoryService | Checks, reserves, and releases stock |
| PaymentService | Processes payments |
| TimerService | Tracks the 15-minute timeout |

## 3. Sequence Diagram

```mermaid
sequenceDiagram
    autonumber
    actor Customer
    participant UI as OrderUI
    participant C as OrderController
    participant I as InventoryService
    participant P as PaymentService
    participant T as TimerService

    Customer->>UI: Place order
    UI->>C: createOrder(itemId, qty)
    C->>I: checkStock(itemId, qty)

    alt Stock available
        I-->>C: stockAvailable
        C->>I: reserveStock(itemId, qty)
        I-->>C: reservationSucceeded
        C->>C: createOrder(Pending)
        C->>T: startTimer(orderId, 15m)
        C-->>UI: Order created
        UI-->>Customer: Show Pending order

        Customer->>UI: Submit payment
        UI->>C: processPayment(orderId, amount)
        C->>P: processPayment(orderId, amount)

        alt Payment successful
            P-->>C: paymentSucceeded
            C->>I: confirmStockDeduction(itemId, qty)
            I-->>C: stockConfirmed
            C->>C: updateStatus(Paid)
            C->>T: cancelTimer(orderId)
            C-->>UI: Payment successful
            UI-->>Customer: Show Paid order
            C->>C: dispatchOrder()
            C->>C: updateStatus(Shipping)
            C->>C: deliveryConfirmed()
            C->>C: updateStatus(Completed)

        else Payment failed
            P-->>C: paymentFailed
            C->>I: releaseStock(itemId, qty)
            I-->>C: stockReleased
            C->>C: updateStatus(Cancelled)
            C-->>UI: Order cancelled
            UI-->>Customer: Show cancellation

        else Payment timeout
            T-->>C: timeoutExpired(orderId)
            C->>I: releaseStock(itemId, qty)
            I-->>C: stockReleased
            C->>C: updateStatus(Cancelled)
            C-->>UI: Order cancelled
            UI-->>Customer: Show timeout
        end

    else Stock unavailable
        I-->>C: stockUnavailable
        C-->>UI: outOfStock
        UI-->>Customer: Show out of stock
    end
```

## 4. Scenario Descriptions

### Scenario 1: Successful Payment

1. Customer places an order.
2. System checks product availability.
3. System reserves stock.
4. Order is created with Pending status.
5. Timer starts for 15 minutes.
6. Customer completes payment successfully.
7. System confirms stock deduction.
8. Timer is cancelled.
9. Order changes to Paid.
10. Order is dispatched and changes to Shipping.
11. Delivery is confirmed and order changes to Completed.

### Scenario 2: Payment Failed

1. Customer places an order.
2. System reserves stock.
3. Order is created with Pending status.
4. Payment processing fails.
5. System releases reserved stock.
6. Order changes to Cancelled.

### Scenario 3: Payment Timeout

1. Customer places an order.
2. System reserves stock.
3. Timer starts for 15 minutes.
4. Customer does not complete payment within 15 minutes.
5. Timer triggers timeoutExpired.
6. System releases reserved stock.
7. Order changes to Cancelled.

### Scenario 4: Stock Unavailable

1. Customer places an order.
2. System checks product availability.
3. Stock is insufficient.
4. System rejects the order.
5. Customer receives an out-of-stock notification.

## 5. Main Business Rules

1. Stock must be available before reservation.
2. Stock must be reserved before payment.
3. A new order starts with Pending status.
4. Payment must succeed within 15 minutes.
5. Failed payment triggers stock release.
6. Timeout triggers automatic stock release.
7. Successful payment changes Order status to Paid.
8. Cancelled orders must not retain reserved stock.

## 6. Conclusion

The sequence diagram illustrates the interaction between the Customer, OrderUI, OrderController, InventoryService, PaymentService, and TimerService.

It describes four main scenarios: successful payment, payment failure, payment timeout, and stock unavailable.

The system ensures proper stock reservation, payment processing, and order status management throughout the order lifecycle.
