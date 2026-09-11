---
tags: [pattern, behavioral]
---

# Command

## 🧠 Problem
SAloon's admin dashboard needs an audit log of every action (booking created, booking cancelled, refund issued) and eventually an "undo last action" button. I don't want to bolt logging/undo onto every single service method separately.

## 🤔 Why does the problem exist
If actions are just direct method calls, there's no *object* representing "the action that happened" — nothing to log, queue, replay, or undo. The action and its execution are fused together.

## 💡 Idea
Wrap each action as a command object with `execute()` (and optionally `undo()`). Now actions can be logged, queued, retried, or reversed as first-class data.

## 🏗️ Structure
```
Command (interface)
   ├── CreateBookingCommand
   ├── CancelBookingCommand
   └── IssueRefundCommand
CommandInvoker — executes commands, keeps history for undo/audit
```

## ☕ Java Implementation
```java
interface Command { void execute(); void undo(); }

class CancelBookingCommand implements Command {
    private Booking booking;
    private BookingState previousState;
    CancelBookingCommand(Booking b) { this.booking = b; }
    public void execute() { previousState = booking.getState(); booking.cancel(); }
    public void undo() { booking.setState(previousState); }
}

class CommandInvoker {
    private Deque<Command> history = new ArrayDeque<>();
    void run(Command c) { c.execute(); history.push(c); logToAudit(c); }
    void undoLast() { if (!history.isEmpty()) history.pop().undo(); }
}
```

## 🏢 SAloon Example
Every admin action goes through `CommandInvoker` — see [[09 - Real Project LLD/SAloon/00 - Booking]]. This is also what powers the audit trail without polluting `BookingService` with logging calls.

## 🎯 When should I use it?
Undo/redo, audit trails, job queues, or decoupling the thing that *triggers* an action (a button, an API call) from the thing that *performs* it.

## ⚠️ When NOT to use it?
If you never need to log, queue, delay, or undo an action, wrapping it as a Command is pure ceremony — just call the method.

## 🔥 Interview Question
*"How would Command support a 'retry failed jobs' feature?"*
→ Since each job is already a `Command` object (not just an executed side effect), failed ones can sit in a retry queue and get `.execute()`'d again later — the pattern gives you a first-class object to persist/requeue in the first place.

## 🔗 Related
[[../Creational/Builder]]
[[09 - Real Project LLD/SAloon/00 - Booking]]

#pattern/behavioral
