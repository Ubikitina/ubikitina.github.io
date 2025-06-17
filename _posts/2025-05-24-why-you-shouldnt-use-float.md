---
layout: post  
title: Why You Shouldn’t Use FLOAT in PostgreSQL (and Other Decimal Precision Gotchas)  
permalink: /2025-05-24-why-you-shouldnt-use-float.html  
date: 2025-05-24
---

As a data engineer, one of the subtlest — and most dangerous — mistakes you can make when designing data models is using the `FLOAT` type to store decimal values when you need precision. It sounds harmless: you just need to save a number with a comma, right?

Well, not quite. The truth is that using `FLOAT` in PostgreSQL (and many other systems) can lead to **silent errors**, **unexpected rounding**, and **data inconsistencies** in critical areas like finance, metrics, or system integrations.

In this post, I’ll explain **why `FLOAT` isn’t your friend**, what kinds of problems it introduces, and what alternatives you should be using if you care about decimal precision.



## What Happens When You Use `FLOAT`?

When you define a column as `FLOAT` (or `DOUBLE PRECISION`, or `FLOAT8`, which are essentially the same), you're using a data type that stores decimal numbers **in binary**, following the IEEE 754 standard.

Here’s the problem: **not all decimal numbers can be represented exactly in binary**. Just like `1/3` has no exact representation in base 10 (`0.333…` infinitely), the number `0.1` can't be precisely stored in binary either. What’s actually stored under the hood is something like:

```
0.10000000000000000555
```

So yes, you insert `0.1`, run a `SELECT`, and you don’t get `0.1` — you get that long mess. At first glance, it might seem harmless, but when performing calculations with many values, these differences **accumulate** and can break your logic.



## Real Errors You Might See

* **Invisible rounding**: The system might show `0.1`, but it's storing something else. This can break comparisons or conditional logic.
* **Cumulative errors in calculations**: Adding up millions of `FLOAT` values may result in totals off by tenths or hundredths.
* **Inconsistencies with external systems**: When integrating with APIs, Excel files, or financial reports, floats can give you subtly incorrect results.


## The Right Alternative: Use `NUMERIC`

PostgreSQL provides a data type called `NUMERIC` (also known as `DECIMAL`) that **stores decimal values exactly**, with no rounding errors.

Example:

```sql
CREATE TABLE payments (
  id SERIAL PRIMARY KEY,
  amount NUMERIC(10, 2)
);
```

Here, you’re specifying that `amount` can have up to 10 digits, with 2 decimal places (e.g., `12345678.90`). This is perfect for:

* Prices, taxes, interest rates
* KPIs and business metrics
* Financial data and accounting
* Any situation where **precision matters**


## What About Performance?

Yes, `NUMERIC` is slightly more expensive in terms of CPU and storage than `FLOAT`, because it's stored as a variable-length decimal structure internally. But in real-world use, **the precision is absolutely worth it**.

If you're working with massive tables and heavy calculations, you can optimize with indexes or materialized views. But if you use `FLOAT`, **you can't fix the precision problem later**. It’s like designing a building with the wrong blueprint — no amount of paint will fix the foundation.


## Extra Technical Nuggets About PostgreSQL and Numbers

Here are a few additional tips and curiosities:

* `FLOAT` without precision defaults to `DOUBLE PRECISION` (`FLOAT8`).
* You can use `pg_typeof(column)` to check the exact type in use.
* PostgreSQL will automatically round if you insert more decimal places than allowed in a `NUMERIC(p, s)` (e.g., `NUMERIC(10,2)`).
* Even though it’s not required, it’s a good practice to **explicitly define precision** in your `NUMERIC` types for clarity and storage control.
* Don’t rely on `ROUND()` to "fix" float values — that just hides the symptoms without addressing the real issue.


## Final Thoughts

If you're working with data that will impact decisions, reports, or important calculations, **avoid using `FLOAT` in PostgreSQL**. This isn’t a minor or theoretical issue — it shows up *in production* and costs time, trust, and money.

**Use `NUMERIC` when you need precision**, and reserve `FLOAT` for scientific or statistical scenarios where minor rounding errors are acceptable.

---

Have you faced `FLOAT`-related issues in your projects? I’d love to hear about it. Feel free to reach out via DM on my [GitHub profile](https://github.com/Ubikitina). 😊💬
