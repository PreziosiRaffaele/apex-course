# Apex OOP Fundamentals
Object-oriented programming (OOP) keeps Apex logic modular and testable. This module reinforces how classes, interfaces, and sharing rules combine to model business behavior.

## Classes, Properties, and Constructors
Define state with properties and inject dependencies through constructors so code stays testable.

```apex
public with sharing class RevenueGoalService {
    private final Decimal target;

    public RevenueGoalService(Decimal targetAmount) {
        target = targetAmount;
    }

    public Boolean isGoalMet(List<Opportunity> pipeline) {
        Decimal total = 0;
        for (Opportunity opp : pipeline) {
            total += opp.Amount;
        }
        return total >= target;
    }
}
```

## Sharing Keywords
- `with sharing` enforces the running user's record visibility.
- `without sharing` elevates to system context (use sparingly).
- `inherited sharing` defers to the caller, ideal for service classes invoked from multiple contexts.

## Inheritance and Polymorphism
Leverage `virtual` and `override` to vary behavior without copy-paste.

```apex
public abstract class DiscountStrategy {
    public abstract Decimal apply(Decimal amount);
}

public virtual class LoyaltyDiscount extends DiscountStrategy {
    public override Decimal apply(Decimal amount) {
        return amount * 0.90;
    }
}

public class SeasonalDiscount extends LoyaltyDiscount {
    public override Decimal apply(Decimal amount) {
        return amount * 0.80;
    }
}
```

## Interfaces and Enums
Interfaces define contracts; enums restrict valid values and serialize cleanly.

```apex
public interface FulfillmentTask {
    void execute();
}

public enum FulfillmentChannel { Warehouse, Dropship, Digital }
```

## Exercise: Strategy Registry
Create a `DiscountRegistry` class that:
1. Stores implementations of `DiscountStrategy` in a `Map<FulfillmentChannel, DiscountStrategy>`.
2. Exposes a method `applyDiscount(FulfillmentChannel channel, Decimal amount)` that returns the adjusted total by delegating to the correct implementation.
3. Defaults to a pass-through strategy when the channel is unmapped.

Deliverable: runnable Apex classes plus an anonymous Apex block demonstrating different channels producing different totals. This mirrors real-world scenarios where revenue policies vary by fulfillment path.
