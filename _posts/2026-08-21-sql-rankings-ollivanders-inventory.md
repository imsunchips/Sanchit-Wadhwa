---
layout: post
title: Returning to SQL - Solving Ollivander's Inventory and the Art of Problem Deconstruction
subtitle: Why Mastering SQL Assessments Is More About Problem Analysis Than Syntax
tags: [sql, data-engineering, databases, problem-solving, career]
author: Sanchit Wadhwa
---

After being away from hands-on SQL for a while, jumping back in to prepare for technical assessments was an eye-opening experience. It quickly became clear how easy it is to forget the nuances—and how much there still is to learn when writing complex analytical queries under test conditions.

To bridge the gap, I decided to work through practical challenges on HackerRank and document the solutions. The biggest takeaway so far? **Solving SQL problems isn't primarily about memorizing syntax; it is about reading comprehension, structured thinking, and deconstructing requirements before writing a single line of code.**

In this post, we will look at how to approach complex SQL problems using a classic challenge: **Ollivander's Inventory**.

---

### The Problem Breakdown

Let's review the requirements from the challenge prompt:

* **Context:** Harry and his friends are at Ollivander's buying a wand for Ron. Hermione wants to find the **minimum number of gold galleons (`coins_needed`)** needed to buy each **non-evil wand** for every combination of **high power** and **age**.
* **Output Required:** `id`, `age`, `coins_needed`, and `power`.
* **Sorting Rule:** Order by `power DESC`, then by `age DESC`.
* **Tables:**
  * `Wands`: `id`, `code`, `coins_needed`, `power`
  * `Wands_Property`: `code`, `age`, `is_evil` (where `0` = non-evil)

---

### Mapping Business Rules to SQL Concepts

When you slow down and analyze the prompt before coding, the technical requirements reveal themselves naturally:

1. *"Non-evil wands"* -> **Filter:** `WHERE is_evil = 0`
2. *"For each age and power combination"* -> **Grouping:** `PARTITION BY age, power`
3. *"Minimum coins needed"* -> **Ranking:** `ORDER BY coins_needed ASC` inside the window function
4. *"Sort result by power and age descending"* -> **Final Presentation:** Outer `ORDER BY power DESC, age DESC`

---

### The Optimal Solution

By using a Common Table Expression (CTE) with `ROW_NUMBER()`, we isolate the logic for picking the cheapest wand per group from the final formatting.

```sql
WITH RankedWands AS (
    SELECT 
        w.id,
        wp.age,
        w.coins_needed,
        w.power,
        ROW_NUMBER() OVER (
            PARTITION BY wp.age, w.power 
            ORDER BY w.coins_needed ASC
        ) AS rowNumber
    FROM Wands w
    JOIN Wands_Property wp ON w.code = wp.code
    WHERE wp.is_evil = 0
)
SELECT 
    id, 
    age, 
    coins_needed, 
    power
FROM RankedWands
WHERE rowNumber = 1
ORDER BY power DESC, age DESC;
```

---

### A 4-Step Framework for Technical SQL Questions

To avoid jumping into code too quickly on future assessments, follow this simple workflow:

1. **Isolate Hard Filters First:** Identify which records should be eliminated upfront (`WHERE` clause).
2. **Define the Grouping Keys:** Determine what combination defines a unique "bucket" (`PARTITION BY`).
3. **Identify the Metric to Optimize:** Figure out what orders items inside each bucket (`ORDER BY` in `OVER()`).
4. **Separate Logic from Presentation:** Compute rankings inside a subquery or CTE, then format and sort the output in the main `SELECT` statement.

---

### Final Thoughts

Getting back into SQL practice has been a great reminder that technical assessments evaluate problem deconstruction just as much as query mechanics. Taking two minutes to break down the requirements on paper almost always saves ten minutes of debugging later.
