---
layout: post
title: "From PySpark to dbt: how I learned dbt from scratch"
permalink: /2026-04-24-learning-dbt-from-scratch.html
date: 2026-04-24
---

A few months ago I changed jobs, and with it, my entire data stack. For years I had been building pipelines on a pure AWS stack: **Glue jobs, plain PySpark, Step Functions and Lambdas**. Now my day to day is **dbt, Snowflake and Airflow**.

This is the first of a few posts about that transition. Not a tutorial, but what it actually felt like to switch stacks as a data engineer. Let's start with learning dbt.

## Where I was coming from

In my previous role, transforming data meant writing PySpark in Glue jobs, orchestrated with Step Functions and a few Lambdas glued in between. A single job could read raw data from **Parquet files** in S3, master data from **DynamoDB** and consumption-ready data from **Aurora**, apply the business transformations, and write the results back to wherever they were needed: Parquet for the gold layer, Parquet or CSV files to be sent somewhere, Aurora or DynamoDB.

Each of those connections had its own connector, quirks and permissions. We had a shared library with common methods so that every read and write was done the same way. It worked, but that overhead pulled attention away from the data itself: as data engineers we also maintained libraries and connectors, and the line between platform work and data engineering wasn't always clear.

In my new role, that changed. A platform team sets up and maintains the infrastructure: the connections live in Airflow, and `dbt run` simply uses them. Everything runs against Snowflake, and exporting or sending files is a separate Airflow job. dbt focuses on one thing: transforming the data inside the warehouse. With that separation of responsibilities, each side can focus on doing its own job well.

## What I thought would be easy

When I looked at dbt for the first time, I was confident. dbt is basically **SQL plus Jinja**, and I already knew both. I had written plenty of SQL, and I had used Jinja templates before. How hard could it be to put them together?

## What was actually hard

It turned out the syntax was the easy part. **The real learning was the mindset.**

Writing a short SQL query is one thing. Writing long models where SQL and Jinja are interleaved, where a `{% raw %}{{ ref('...') }}{% endraw %}` is not just a table name but a dependency in a graph, where a macro generates half of your query, is something else. I had to stop thinking in scripts and start thinking in **models**:

- **Every model is a `SELECT`.** You don't write `CREATE TABLE` or `INSERT`: you describe what the data should look like, and dbt decides how to build it.
- **`ref()` builds the DAG for you.** Instead of orchestrating steps by hand, dependencies come from the models themselves.
- **Materializations are a decision, not code.** Table, view, incremental or ephemeral is a line of configuration, not a rewrite.
- **Jinja is for logic, SQL is for data.** The real skill is knowing when a macro helps and when plain SQL is clearer.

It was a paradigm shift: less "how do I run this" and more "what should this data be".

There wasn't a single "aha" moment. It was gradual, but two moments made a real difference.

The first one came very early. I opened a long model written by someone else, full of Jinja, and couldn't follow it at all. Then I ran `dbt compile` and looked at the SQL it generated. Suddenly it made sense: **Jinja doesn't run anything, it just writes SQL for you.** Once I saw the compiled output, the templating started feeling like a tool.

Here's a simplified example of what I mean. A model like this, with a Jinja loop and a `ref()`:

```sql
-- models/order_payments.sql
{% raw %}{% set payment_methods = ['card', 'bank_transfer', 'gift_card'] %}

select
    order_id,
    {% for method in payment_methods %}
    sum(case when payment_method = '{{ method }}' then amount end) as {{ method }}_amount
    {%- if not loop.last %},{% endif %}
    {% endfor %}
from {{ ref('stg_payments') }}
group by order_id{% endraw %}
```

compiles into plain SQL:

```sql
-- target/compiled/.../order_payments.sql
select
    order_id,
    sum(case when payment_method = 'card' then amount end) as card_amount,
    sum(case when payment_method = 'bank_transfer' then amount end) as bank_transfer_amount,
    sum(case when payment_method = 'gift_card' then amount end) as gift_card_amount
from analytics.dbt_ubikitina.stg_payments
group by order_id
```

The loop is gone, and `ref('stg_payments')` has become a real table name. That's all Jinja does.

The second moment came halfway through. In the code I had inherited, I found the same logic duplicated in several models, so a single change meant editing the same thing in several files. That was inefficient, and it's what pushed me to start using macros more. Seeing those models get shorter and cleaner, with the logic defined in one place, was when I understood what Jinja is really for in dbt.

## What helped me

What helped me the most was a Udemy course: **[The Complete dbt (Data Build Tool) Bootcamp: Zero to Hero](https://www.udemy.com/course/complete-dbt-data-build-tool-bootcamp-zero-to-hero-learn-dbt/)**, by Zoltan C. Toth and Miklos Petridisz. It is built around a hands-on project with Airbnb data, so you go from sources to models, tests and documentation in a real-looking project, instead of isolated examples.

Two parts of it made the biggest difference for me:

- **The core hands-on sections**, from building the first version of the project up to the "dbt Hero" milestone: models, materializations, sources and seeds, snapshots, tests, Jinja and macros, documentation, and analyses, hooks and exposures. That's where the dbt way of thinking really clicked, because you build every piece yourself and see how it fits in the DAG.
- **The theory sections**: the data maturity model, data warehouses vs data lakes vs lakehouses, the modern data stack and slowly changing dimensions. Coming from a Spark-on-AWS world, they gave me the context to understand *why* dbt works the way it does, and where it fits in a modern data platform.

The rest of the course (CI, orchestration, certification prep…) is useful too, but those two blocks were what took me from zero to productive.

## Why the learning curve is gentler than PySpark

Even with that mindset shift, looking back I think dbt is easier to learn than PySpark, and not only because SQL is more widely known.

With PySpark you have to understand distributed computing to write good code: partitions, shuffles, memory, cluster sizing. Optimizing a job often means going down to a very low level of the infrastructure behind it: calling `repartition()` or `coalesce()` to control how many partitions you write, caching DataFrames at the right moment, broadcasting small tables to avoid a shuffle, tuning the number and type of workers.

With dbt on top of Snowflake, most of that disappears from your code. You (or your platform team) pick a warehouse size, like choosing a T-shirt size, and **Snowflake decides what runs behind it**. You don't decide whether 5, 6 or 8 machines are spun up, and you don't sprinkle `coalesce()` calls to squeeze out performance. You write the model, run dbt, and that's it. The performance side is much more transparent to you as a developer.

That doesn't mean performance doesn't matter. It means you can be productive much earlier, learn the optimization side as you go, and **spend most of your energy on the data itself rather than on the infrastructure behind it**. In the long run, I think that's what leads to better results.

## Tips if you're starting with dbt

- **Don't underestimate it because you know SQL.** The SQL is the easy part; the structure is the real learning.
- **Learn `ref()` and sources first.** Everything else is built on top of the DAG.
- **Start with plain SQL, and reach for a macro when you see the same logic repeated across models.** That's when macros really pay off.
- **Do a full project, not just snippets.** A course or side project with real-looking data makes the concepts stick.
- **When a model full of Jinja looks like magic, run `dbt compile`.** Reading the SQL it generates is the fastest way to understand what's going on.

## An open question

One question kept coming back to me after a few weeks with dbt: **is it actually faster than Glue?** My first impression was a clear yes. The real answer turned out to be more interesting, but that deserves its own post.
