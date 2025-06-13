# Lifecycle Management

## Transition to Secure

## Transition to RMA

```mermaid
stateDiagram
    [*]              --> VIRGIN
    VIRGIN           --> NORMAL_PROVISION
    NORMAL_PROVISION --> SECURE
    NORMAL_PROVISION --> SECURE_W_DEBUG
    SECURE           --> DEAD
    SECURE           --> RMA
    RMA              --> OpenRMA
```
