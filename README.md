# SQL Interview Patterns

A structured collection of SQL interview problems
organized by **patterns and sub-patterns**.

This repository follows a **pattern → sub-pattern → questions**
approach to solve SQL problems commonly asked in
Data Science and Analytics interviews.

---

## Repository Structure

```text
sql-interview-patterns/
│
├── 01_where/
│   ├── hardcoded_filter/
│   ├── subquery_filter/
│   ├── in_not_in/
│   ├── exists_not_exists/
│   ├── null_handling/
│   └── column_vs_column/
│
├── 02_order_by/
│   ├── single_column/
│   ├── derived_column/
│   ├── multi_column/
│   └── order_by_position/
│
├── 03_limit/
│   ├── limit_basic/
│   ├── limit_with_order_by/
│   ├── top_sql_server/
│   └── fetch_first/
│
├── 04_joins/
│   ├── inner_join/
│   ├── left_join/
│   ├── left_join_null/
│   ├── self_join/
│   └── multi_table_join/
│
├── 05_group_by/
│   ├── single_column_group/
│   ├── multi_column_group/
│   ├── group_by_with_join/
│   └── date_based_group/
│
├── 06_having/
│   ├── count_having/
│   ├── sum_avg_having/
│   └── having_vs_where/
│
├── 07_aggregates/
│   ├── count/
│   ├── count_distinct/
│   ├── sum_avg/
│   └── min_max/
│
├── 08_window_functions/
│   ├── row_number/
│   ├── rank_dense_rank/
│   ├── running_total/
│   ├── top_k_per_group/
│   └── lead_lag/
│
└── 09_misc/
    ├── case_when/
    ├── date_functions/
    ├── string_functions/
    └── union_vs_union_all/

## Notes
- Each **pattern folder** contains sub-pattern folders
- Each **sub-pattern folder** contains interview-style SQL questions
- Explanations are included inside individual pattern folders
- Primary SQL dialect used: **MySQL**

