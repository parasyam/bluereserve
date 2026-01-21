# Blu-Reserve Seat Allocation System

## 1. Problem Recap (Why this exists)

- Office seating is limited, especially during lunch hours.
- Employees bring or orders different foods
- We need to:
  - Allocate seats fairly
  - Notify cooking team at the *right time*
  - Notify employees *before* their seat is ready
  - Avoid food waste and confusion

This is **not just booking**. This is **time-synchronized resource allocation**.

---

## 2. Key Concepts (Simple Definitions)

### Cooking Time

Time required to prepare food:

- Food from home → **0 min**
- Beverages → **10 min**
- Meals → **20 min**

### Seat Time

- Time after which a seat becomes free
- Chosen by the employee currently occupying the seat

---

## 3. Roles Involved

### Employee

- Books a seat
- Selects food type
- Optionally requests extra seat time

### Cooking Department

- Prepares food only when notified

### System

- Decides **when** to allocate seats
- Decides **when** to notify cooking & employee

---

## 4. Data Structures Used (Core of the Algorithm)

### Seating Heap (Min Heap)

- Stores: `(seat_number, remaining_seat_time)` i.e seat number is mapped with seat time
- Top element → seat that will free **earliest**

### Employee Heap (Min Heap)

- Stores: `(employee_id, cooking_time)` i.e employee id is mapped with cooking time
- Top element → employee with **least cooking time**

Why heaps?

- We always need the *next available seat* and the *fastest-preparing employee*.

---

## 5. Seat Allocation Strategy

### Phase 1: Seats Available (FCFS)

- If seats are empty:
  - Allocate seat immediately
  - Follow **First Come First Serve**
  - Add seat info to **Seating Heap**
    (seat number mapped to seat time is added to seating heap)

---

### Phase 2: All Seats Filled

- New employee requests:
  - Add employee to **Employee Heap**
  - Do NOT allocate seat immediately

For every 2 minutes:

- Pick:
  - Seat with **least remaining seat time** (Seating Heap top)
  - Employee with **least cooking time** (Employee Heap top)
- Map them together

i.e lowest seat-time seat is mapped to employee with lowest cooking time.

---

## 6. Notification Logic (Critical Part)

*(Cooking time = time required to cook food)*

### General Rules

- Cooking team should be notified **exactly cooking_time minutes** before seat frees
- Employee should be notified **10 minutes before seat is free**

---

## 7. Case-Based Flow

### Case 1: Employee A does NOT request extra time

**Definitions:**

- Employee A → currently using the seat
- Employee B → next employee waiting
- Seat Time → remaining time for A
- Cooking Time → time required to cook item ordered by B

#### Condition

- `seat_time == cooking_time`

#### Actions

- Notify cooking team immediately
- Notify Employee B **10 minutes before food is ready**:
  - Seat number
  - "Come in 10 minutes"

**Result:** Perfect sync, no waiting, no food waste

---

### Case 2: Employee A requests extra time

#### Subcase 2.1: `seat_time < cooking_time`

- Cooking team already notified
- Food preparation has started

➡ Reject extension request

**Reason:** Seat delay would cause prepared food to wait

---

#### Subcase 2.2: `seat_time > cooking_time`

- Cooking team NOT yet notified

➡ Allow extension

- Add requested time to seat_time
- Update Seating Heap

---

## 8. Manager Time-Used Constraint & Billing Logic

### Time-Used Constraint (Manager Level)

- Each manager has a **time-used constraint** that tracks total seat minutes consumed by their team.
- For every employee seat usage:
  - Actual seat usage time (in minutes) is recorded.
  - These minutes are **added to the manager’s total time-used counter**.

### Token Deduction Logic

- Blu Dollars / tokens are deducted **based on seat minutes used**, not just booking count.
- Formula (conceptual):
  - `tokens_deducted = seat_minutes_used × cost_per_minute`
- Deduction happens:
  - Incrementally as time passes **or**
  - Once the employee completes their seat usage (implementation choice).

### Seat Completion Update

- When an employee finishes using a seat:
  - Their total seat time is finalized.
  - That seat time is:
    - Added to the **manager’s time-used constraint**
    - Used for final token deduction
  - The seat is marked free and pushed back into the **Seating Heap**.

This ensures:
- Fair billing based on real usage
- No free overuse of seats
- Clear accountability at manager level

---

## 9. Why This Algorithm Works Well

- Prevents food wastage
- Minimizes employee waiting time
- Fair seat allocation
- Scales well (heap operations are log(n))
- Clean separation of concerns

This is **event-driven**, not time-polling.

---

## 10. High-Level Flow Summary

1. Employee books seat & selects food
2. System calculates cooking time
3. If seat free → allocate immediately
4. If not → add to Employee Heap
5. Match earliest seat with fastest cooking employee
6. Send notifications at exact moments
