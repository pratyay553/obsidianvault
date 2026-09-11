---
tags: [lld-problem]
---

# Splitwise (Expense Sharing)

## 🧠 Problem
Design an expense-splitting app: add an expense, split it among a group (equally, by percentage, or by exact amount), and show simplified "who owes whom" balances.

## 🤔 Why does the problem exist
The interesting part isn't recording expenses — it's that the *splitting rule* varies (equal/percentage/exact) and needs validation per rule (percentages must sum to 100%, exact amounts must sum to the total), plus balance simplification is a genuine minimal-cash-flow graph problem.

## 💡 Idea
Each split type is its own `SplitStrategy` that validates and computes individual shares. Balances are tracked as a graph (who-owes-whom), simplified by netting out debts.

## 🏗️ Structure
```
SplitStrategy (interface) → EqualSplit, PercentageSplit, ExactSplit
Expense — payer, amount, participants, splitStrategy
BalanceSheet — Map<Pair<User,User>, Double> net balances
```

## ☕ Java Implementation
```java
interface SplitStrategy { Map<User, Double> computeShares(double amount, List<User> users, Map<User, Double> inputs); }

class PercentageSplit implements SplitStrategy {
    public Map<User, Double> computeShares(double amount, List<User> users, Map<User, Double> pct) {
        if (pct.values().stream().mapToDouble(Double::doubleValue).sum() != 100.0)
            throw new IllegalArgumentException("percentages must sum to 100");
        Map<User, Double> shares = new HashMap<>();
        pct.forEach((u, p) -> shares.put(u, amount * p / 100));
        return shares;
    }
}
```

## 🏢 SAloon Tie-in
`SplitStrategy` is structurally identical to SAloon's `PricingStrategy` — see [[../05 - Design Patterns/Behavioral/Strategy]] and [[../09 - Real Project LLD/SAloon/04 - Pricing]]. Same pattern, different domain rule.

## Patterns Applied
- [[../05 - Design Patterns/Behavioral/Strategy]] — split types
- [[../05 - Design Patterns/Behavioral/Observer]] — notify participants when a new expense is added

## 🔥 Interview Question
*"How do you simplify balances so Alice doesn't pay Bob who then pays Charlie for the same net debt?"*
→ Compute each person's net balance (total owed minus total owing), then greedily match the largest creditor with the largest debtor repeatedly — minimizes the number of actual transactions needed to settle everyone up.

## 🔗 Related
[[../05 - Design Patterns/Behavioral/Strategy]]
[[../09 - Real Project LLD/SAloon/04 - Pricing]]

#lld-problem
