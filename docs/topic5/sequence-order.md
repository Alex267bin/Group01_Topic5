# Sequence Diagram - Order Processing

## 1. Overview

This diagram describes the order processing workflow
in the Flash Sale E-commerce system.

It includes:
- Successful order and payment.
- Payment failure.
- Payment timeout after 15 minutes.
- Automatic stock release.

## 2. Sequence Diagram

```mermaid
sequenceDiagram
    autonumber

    actor Customer
    participant UI as OrderUI
    participant C as OrderController
    participant I as InventoryService
    participant P as PaymentService
    participant T as TimerService

    Customer->>UI: createOrderRequest()
    UI->>C: checkout(itemId, qty, amount)

    C->>I: reserveStock(itemId, qty)

    alt Stock available
        I-->>C: reservationSucceeded()

        C->>C: new Order(status=Pending)
        C->>T: startTimer(orderId, 15m)
        C->>P: processPayment(orderId, amount)

        alt Payment successful within 15 mins
            P-->>C: paymentSucceeded
            C->>I: confirmStockDeduction(itemId, qty)
            I-->>C: stockConfirmed()
            C->>C: updateStatus(Paid)
            C-->>UI: orderConfirmed(orderId)
            UI-->>Customer: showSuccess()

        else Payment failed
            P-->>C: paymentFailed
            C->>I: releaseStock(itemId, qty)
            I-->>C: stockReleased()
            C->>C: updateStatus(Cancelled)
            C-->>UI: orderCancelled()
            UI-->>Customer: showFailure()

        else Timeout after 15 minutes
            T-->>C: timeoutExpired(orderId)
            C->>I: releaseStock(itemId, qty)
            I-->>C: stockReleased()
            C->>C: updateStatus(Cancelled)
            C-->>UI: orderCancelled()
            UI-->>Customer: showTimeout()
        end

    else Stock unavailable
        I-->>C: reservationFailed()
        C-->>UI: outOfStock()
        UI-->>Customer: showOutOfStock()
    end
```

## 3. Participants

| Participant | Responsibility |
|---|---|
| Customer | Initiates an order |
| OrderUI | Receives customer requests |
| OrderController | Controls order processing |
| InventoryService | Reserves and releases stock |
| PaymentService | Processes payments |
| TimerService | Tracks the 15-minute timeout |

## 4. Main Business Rules

1. Stock must be reserved before payment.
2. A new order starts with Pending status.
3. Payment must succeed within 15 minutes.
4. Failed payment triggers stock release.
5. Timeout triggers automatic stock release.
6. Successful payment changes Order status to Paid.
7. Cancelled orders must not retain reserved stock.
