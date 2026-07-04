# ⚽ Football Ticket Booking System — Database Design & SQL Queries

Welcome to the **Football Ticket Booking System** database project. This repository contains the complete database design, Entity-Relationship Diagram (ERD), schema definitions, and a suite of analytical SQL queries tailored for managing a football tournament ticket sales system.

---

## 📊 Entity Relationship Diagram (ERD)

ERD diagram live link (drawio): https://drive.google.com/file/d/1hnaOeTOyFuxKhUptxr7JFXaXdH0NOlVh/view?usp=sharing

Below is the Entity-Relationship Diagram (ERD) representing the database schema design:

<p align="center">
  <img src="asset/ERD - Football Ticket Booking System - B7A3.drawio.png" alt="Entity Relationship Diagram (ERD)" width="90%" style="border-radius: 8px; box-shadow: 0 4px 10px rgba(0,0,0,0.15);"/>
</p>

---

## 🗄️ Database Schema & Architecture

The database is designed with three core tables, maintaining strict data integrity through foreign keys, primary keys, and field-level domain constraints (`CHECK` constraints).

### 1. `Users` Table

Stores information about the system's users, who can either be Ticket Managers or Football Fans.

- **`user_id`** (INT, Primary Key): Unique identifier for each user.
- **`full_name`** (VARCHAR(100), Not Null): Full name of the user.
- **`email`** (VARCHAR(100), Unique, Not Null): Email address, must be unique across the system.
- **`role`** (VARCHAR(50), Not Null): Role of the user. Restricted via a `CHECK` constraint to:
  - `Ticket Manager`
  - `Football Fan`
- **`phone_number`** (VARCHAR(50), Null): User's contact number (optional).

### 2. `Matches` Table

Contains details of the scheduled football fixtures, their pricing, and ticket availability.

- **`match_id`** (INT, Primary Key): Unique identifier for each match.
- **`fixture`** (VARCHAR(100), Not Null): Teams playing (e.g., 'Real Madrid vs Barcelona').
- **`tournament_category`** (VARCHAR(50), Not Null): The competition/league (e.g., 'Champions League').
- **`base_ticket_price`** (DECIMAL(10,2), Not Null): Price of a single ticket. Regulated to be positive:
  - `CHECK (base_ticket_price > 0)`
- **`match_status`** (VARCHAR(50), Not Null): Current status of the match. Restricted via a `CHECK` constraint to:
  - `Available`
  - `Selling Fast`
  - `Sold Out`
  - `Postponed`

### 3. `Bookings` Table

A junction table representing the tickets booked by users for specific matches.

- **`booking_id`** (INT, Primary Key): Unique identifier for each booking.
- **`user_id`** (INT, Not Null): Foreign Key referencing `Users(user_id)`.
- **`match_id`** (INT, Not Null): Foreign Key referencing `Matches(match_id)`.
- **`seat_number`** (VARCHAR(10), Null): Designated seat code (optional before final ticket printing).
- **`payment_status`** (VARCHAR(20), Null): Payment state. Restricted via a `CHECK` constraint to:
  - `Pending`
  - `Confirmed`
  - `Cancelled`
  - `Refunded`
- **`total_cost`** (DECIMAL(10,2), Not Null): Total booking amount paid/payable. Regulated to be non-negative:
  - `CHECK (total_cost >= 0)`

---

## ⚡ Core SQL Queries ([./queries.sql])

### 🔍 Query 1: Filter Available Champions League Matches

- **Goal**: Retrieve all upcoming football matches belonging to the 'Champions League' where the tickets are still `'Available'`.
- **SQL Implementation**:

```sql
SELECT
    match_id,
    fixture,
    base_ticket_price
FROM Matches
WHERE tournament_category = 'Champions League'
  AND match_status = 'Available'
ORDER BY match_id ASC;
```

---

### 🔍 Query 2: Wildcard Case-Insensitive User Search

- **Goal**: Search for all users whose full names start with `'Tanvir'` or contain the phrase `'Haque'` (matching case-insensitively).
- **SQL Implementation**:

```sql
SELECT
    user_id,
    full_name,
    email
FROM Users
WHERE full_name LIKE 'Tanvir%'
   OR full_name ILIKE '%Haque%'
ORDER BY user_id ASC;
```

---

### 🔍 Query 3: Handling Missing Payment Statuses

- **Goal**: Retrieve all booking records where the payment status is missing (`NULL`), replacing the empty result with `'Action Required'` for operational visibility.
- **SQL Implementation**:

```sql
SELECT
    booking_id,
    user_id,
    match_id,
    COALESCE(payment_status, 'Action Required') AS systematic_status
FROM Bookings
WHERE payment_status IS NULL;
```

---

### 🔍 Query 4: Multi-Table Join for Ticket Details

- **Goal**: Retrieve match booking details combined with the User's full name and the scheduled Match fixture teams.
- **SQL Implementation**:

```sql
SELECT
    b.booking_id,
    u.full_name,
    m.fixture,
    b.total_cost
FROM Bookings b
INNER JOIN Users u
    ON u.user_id = b.user_id
INNER JOIN Matches m
    ON m.match_id = b.match_id
ORDER BY b.booking_id ASC;
```

---

### 🔍 Query 5: Full Outer Join for User Booking Status

- **Goal**: Display a comprehensive list of all users and their booking IDs, ensuring that fans who have never bought a ticket are still listed.
- **SQL Implementation**:

```sql
SELECT
    u.user_id,
    u.full_name,
    b.booking_id
FROM Users u
FULL JOIN Bookings b
    ON u.user_id = b.user_id
ORDER BY u.user_id ASC;
```

---

### 🔍 Query 6: Subquery Filter for High-Value Bookings

- **Goal**: Find all ticket bookings where the total cost is strictly higher than the average cost of all bookings.
- **SQL Implementation**:

```sql
SELECT
    booking_id,
    match_id,
    total_cost
FROM Bookings
WHERE total_cost > (
    SELECT ROUND(AVG(total_cost), 2)
    FROM Bookings
)
ORDER BY booking_id ASC;
```

---

### 🔍 Query 7: Sorting and Pagination (Offset Limit)

- **Goal**: Retrieve the top 2 most expensive matches sorted by base ticket price, skipping the absolute highest premium match.
- **SQL Implementation**:

```sql
SELECT
    match_id,
    fixture,
    base_ticket_price
FROM Matches
ORDER BY base_ticket_price DESC
LIMIT 2 OFFSET 1;
```

---
