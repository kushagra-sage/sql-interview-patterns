## 1. What is the WHERE clause?

The `WHERE` clause is used to **filter rows** from a table (or joined tables) based on specified conditions.

Only rows that satisfy the condition in the `WHERE` clause are passed to the next stages of the query.

In practice, `WHERE` answers one fundamental question:

> **Which rows should be considered?**

---

## 2. Logical Execution Order of SQL (Essential Foundation)

SQL queries are **not executed in the order they are written**.

The logical execution order is:

1. `FROM`
2. `JOIN`
3. `WHERE`
4. `GROUP BY`
5. `HAVING`
6. `SELECT`
7. `ORDER BY`
8. `LIMIT`

### Key implication

* `WHERE` operates on **individual rows**
* Aggregated values do **not exist yet** when `WHERE` runs

This is why aggregate functions cannot be used in `WHERE`.

---

## 3. WHERE vs HAVING (Conceptual Difference)

| Aspect                      | WHERE | HAVING |
| --------------------------- | ----- | ------ |
| Operates on                 | Rows  | Groups |
| Runs before aggregation     | Yes   | No     |
| Can use aggregate functions | No    | Yes    |

Example logic difference:

* Filter rows → `WHERE`
* Filter aggregated results → `HAVING`

This separation exists because aggregation happens **after** row-level filtering.

---

## 4. General Structure of WHERE

```sql
SELECT columns
FROM table
WHERE condition;
```

A `condition` evaluates to **TRUE**, **FALSE**, or **UNKNOWN**.

Only rows where the condition evaluates to **TRUE** are kept.

---

## 5. Boolean Logic in WHERE

The `WHERE` clause uses boolean expressions combined with:

* `AND`
* `OR`
* `NOT`

### Evaluation rules

* `AND` → all conditions must be TRUE
* `OR` → at least one condition must be TRUE
* `NOT` → negates a condition

Parentheses should always be used for clarity when mixing `AND` and `OR`.

---

## 6. Comparison Operators

Common comparison operators used in `WHERE`:

* `=`  (equal)
* `!=` or `<>` (not equal)
* `>`  (greater than)
* `<`  (less than)
* `>=`
* `<=`

These operators compare:

* column vs value
* column vs column
* column vs expression

---

## 7. Truth Values and Three-Valued Logic

SQL uses **three-valued logic**:

* TRUE
* FALSE
* UNKNOWN (usually caused by `NULL`)

Important rule:

* Rows evaluating to `UNKNOWN` are **not returned**

This behavior is the root cause of many subtle bugs in SQL.

---

## 8. NULL in WHERE (High-Impact Concept)

`NULL` represents **missing or unknown data**.

### Important rules:

* `NULL = NULL` → UNKNOWN
* `NULL != NULL` → UNKNOWN
* Any comparison with `NULL` → UNKNOWN

Correct way to handle NULL:

```sql
IS NULL
IS NOT NULL
```

Incorrect:

```sql
= NULL
!= NULL
```

---

## 9. Categories of WHERE Conditions

In practice, WHERE conditions fall into **distinct logical categories**.
This repository organizes them as **sub-patterns**.

At a high level, WHERE conditions are used to:

1. Match fixed values
2. Compare against computed results
3. Check membership in a set
4. Check existence of related records
5. Handle missing data
6. Compare values across columns or expressions

Each of these has different behavior, pitfalls, and use cases.

---

## 10. Why WHERE Deserves Special Attention

Most SQL errors do **not** come from syntax.
They come from:

* Incorrect filtering
* Filtering at the wrong stage
* Misunderstanding NULL behavior
* Using the wrong logical construct

A correct `WHERE` clause simplifies everything that comes after it.

## 11. Fixed-Value Filtering (Hardcoded Conditions)

This is the simplest and most direct use of `WHERE`.

A column is compared against a known, fixed value.

Typical comparisons:

* equality or inequality
* numeric thresholds
* categorical values

These conditions are deterministic and do not depend on other rows.

### Characteristics

* Evaluated independently for each row
* No dependency on subqueries or other tables
* Very predictable behavior

### Conceptual role

Fixed-value filters reduce the dataset to a **relevant subset** before any further logic is applied.

---

## 12. Subquery-Based Filtering

In this form, a row is filtered based on the result of another query.

The subquery can return:

* a single scalar value
* a set of values
* a derived threshold

### Why this exists

Some conditions cannot be evaluated using a constant value because:

* the threshold depends on the data itself
* the comparison must adapt dynamically

Examples of conceptual use:

* comparing against an average
* matching maximum or minimum values
* filtering based on derived metrics

### Important property

Subqueries in `WHERE` are evaluated **before grouping** but **after joins**.

---

## 13. Set-Based Filtering (IN / NOT IN)

Set-based filtering checks whether a value belongs to a set of values.

The set can be:

* explicitly listed
* produced by a subquery

### Conceptual use

* Membership testing
* Category inclusion or exclusion
* Matching foreign keys across datasets

### Critical behavior

When a subquery used with `IN` or `NOT IN` contains `NULL`, the result may become **UNKNOWN**.

This is not a syntax issue but a logical consequence of SQL’s three-valued logic.

---

## 14. Existence-Based Filtering (EXISTS / NOT EXISTS)

Existence-based filtering checks whether **at least one matching row exists** in a related dataset.

Unlike `IN`, it does not compare values directly.

### Key distinction

* `IN` compares values
* `EXISTS` checks for row presence

### Why this pattern exists

Some problems are not about matching values but about **relationships**:

* whether related data exists
* whether related data does not exist

This makes `EXISTS` conceptually closer to joins than to simple filtering.

---

## 15. NULL-Sensitive Filtering

NULL represents missing or unknown information.

Filtering with NULL behaves differently from all other comparisons.

### Core rules

* Comparisons involving NULL do not evaluate to TRUE or FALSE
* They evaluate to UNKNOWN
* Rows with UNKNOWN conditions are excluded

### Why special handling is required

Because NULL is not a value, it cannot be compared using standard operators.

This is why SQL provides:

* `IS NULL`
* `IS NOT NULL`

NULL handling often changes the meaning of a query entirely.

---

## 16. Column-to-Column and Expression-Based Filtering

In this form, the filter compares:

* one column to another
* a column to a derived expression

This pattern is common when:

* the condition depends on relative values
* timing or sequencing matters
* comparisons involve calculated intervals

### Conceptual role

These filters express **relationships within a row**, not absolute conditions.

They are especially common in analytical queries where time, order, or progression matters.

---

## 17. Combining Multiple WHERE Conditions

Real-world filtering rarely involves a single condition.

Conditions are combined using boolean logic.

### Important considerations

* Operator precedence (`AND` before `OR`)
* Explicit grouping using parentheses
* Short-circuit behavior

Incorrect grouping of conditions is one of the most common causes of incorrect results.

---

## 18. WHERE with JOINs

When joins are involved, `WHERE` conditions can apply to:

* the primary table
* the joined table
* both

### Conceptual distinction

* Join condition (`ON`) defines how tables relate
* WHERE condition defines which resulting rows are kept

Placing a condition in `ON` vs `WHERE` can change the outcome of a query, especially with outer joins.

---

## 19. WHERE as a Performance Boundary

Although this document focuses on logic, `WHERE` also plays a role in performance.

Early and selective filtering:

* reduces intermediate data
* simplifies downstream operations
* lowers computational cost

Conceptually, `WHERE` acts as a **boundary** that limits how much data flows into later stages.

---

## 20. Mapping Theory to Sub-Patterns

The theoretical categories discussed here map directly to the sub-pattern folders:

| Conceptual Category      | Sub-Pattern Folder  |
| ------------------------ | ------------------- |
| Fixed-value filtering    | `hardcoded_filter`  |
| Subquery-based filtering | `subquery_filter`   |
| Set membership           | `in_not_in`         |
| Existence checks         | `exists_not_exists` |
| NULL handling            | `null_handling`     |
| Column comparisons       | `column_vs_column`  |

This structure reflects **how SQL filtering is actually used**, not just how it is written.

---


## 21. WHERE and Three-Valued Logic (Deep Dive)

SQL does not use simple boolean logic.
It uses **three-valued logic**:

* TRUE
* FALSE
* UNKNOWN

A row is returned **only if the WHERE condition evaluates to TRUE**.

### Implication

* FALSE → row excluded
* UNKNOWN → row excluded

This is why many queries “look correct” but return fewer rows than expected.

---

## 22. Common Sources of UNKNOWN Results

UNKNOWN most often appears due to:

* Comparisons with `NULL`
* `NOT IN` with subqueries containing `NULL`
* Complex boolean expressions with `OR`

Understanding where UNKNOWN comes from is essential to writing correct filters.

---

## 23. WHERE with NOT (Negation)

Negation in SQL is not symmetric.

For example:

* `NOT (A = B)` is **not equivalent** to `A != B` when `NULL` is involved

Because:

* `A = NULL` → UNKNOWN
* `NOT UNKNOWN` → UNKNOWN

Negation preserves UNKNOWN rather than flipping it.

This behavior is often unintuitive but fully consistent with SQL logic.

---

## 24. WHERE and NOT IN vs NOT EXISTS

This distinction deserves special attention.

### NOT IN

* Compares values
* Fails silently when `NULL` is present
* Can eliminate all rows unexpectedly

### NOT EXISTS

* Checks row existence
* Ignores `NULL` values naturally
* Produces predictable results

Because of this, existence-based filtering is generally safer when relationships are involved.

---

## 25. WHERE with OUTER JOINS

Filtering after an outer join can unintentionally turn it into an inner join.

### Why this happens

* Rows introduced by an outer join contain `NULL`
* A WHERE condition on the joined table removes those rows

This changes the meaning of the query.

### Conceptual rule

* Use `ON` to control join behavior
* Use `WHERE` to control final row selection

Mixing these responsibilities leads to subtle bugs.

---

## 26. WHERE vs ON (Conceptual Separation)

Although both restrict rows, they serve different purposes:

* `ON` defines **how tables relate**
* `WHERE` defines **which rows survive**

Placing a condition in the wrong place can change the semantics without causing any syntax error.

---

## 27. WHERE and Aggregation Boundaries

Since `WHERE` runs before `GROUP BY`:

* It cannot see aggregated results
* It affects which rows participate in aggregation

Filtering too early can remove rows that should contribute to a group.

Filtering too late can inflate aggregates.

Understanding this boundary is essential for correct analytical queries.

---

## 28. WHERE with Window Functions

Window functions are evaluated **after WHERE**.

This means:

* WHERE cannot reference window function results
* Filtering on window outputs requires an outer query

Conceptually:

* WHERE controls the input
* Window functions operate on the filtered dataset

This separation enforces a clear data flow.

---

## 29. Order of Conditions Inside WHERE

SQL engines may reorder conditions for optimization, but **logical correctness depends on equivalence**.

Important practices:

* Avoid relying on short-circuit behavior
* Write conditions that are logically independent
* Use parentheses to make intent explicit

Readability directly affects correctness in complex filters.

---

## 30. WHERE as a Declarative Constraint

The WHERE clause does not describe *how* filtering is done.

It describes *what conditions must be true*.

The database engine decides:

* evaluation order
* indexing strategy
* execution plan

Because SQL is declarative, correctness matters more than control flow.

---

## 31. Typical Logical Mistakes

Most incorrect queries fail due to:

* Incorrect NULL assumptions
* Wrong placement of filters
* Overuse of NOT
* Confusing row-level and group-level conditions
* Misunderstanding join boundaries

These mistakes do not produce errors — they produce **wrong answers**.

---

## 32. How to Reason About WHERE Correctly

Before writing SQL, mentally answer:

1. What rows exist after FROM and JOIN?
2. Which of those rows should be kept?
3. Could any values be NULL?
4. Is this condition row-level or group-level?
5. Does this filter belong before or after aggregation?

This reasoning prevents most errors.

---

## 33. WHERE as the Foundation of Query Correctness

Every SQL query builds on the correctness of its WHERE clause.

If WHERE is wrong:

* Aggregation is wrong
* Ranking is wrong
* Analysis is wrong

A precise WHERE clause simplifies all later logic.

---

## Final Perspective

The WHERE clause is not just a filter.
It defines the **logical boundary of your dataset**.

Understanding:

* when it runs
* what it can see
* how it handles NULL
* how it interacts with joins and aggregation

is essential for writing correct and reliable SQL.

---

Each sub-pattern directory applies the concepts above
to concrete SQL problems, focusing on one logical idea at a time.

