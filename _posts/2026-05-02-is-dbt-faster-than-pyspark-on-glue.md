---
layout: post
title: "Is dbt on Snowflake faster than PySpark on Glue? Not exactly the right question"
permalink: /2026-05-02-is-dbt-faster-than-pyspark-on-glue.html
date: 2026-05-02
---

After a few weeks with dbt, I had a clear impression: it felt much faster than what I was used to.

In my previous role, our transformations ran as PySpark jobs on AWS Glue. Depending on the job and how much it had been tuned, even jobs working on small tables could take up to **10 minutes**. Now, with dbt on Snowflake, models processing many more rows finish in a few minutes, in my experience.

So, is dbt on Snowflake faster than PySpark on Glue? The honest answer is: it depends, and it's not quite the right question. dbt doesn't run anything itself: it compiles models into SQL and sends them to Snowflake. The real comparison is between two engines, Spark on Glue and Snowflake, and how each of them starts up and handles small jobs. Let me explain.

## Spark on Glue: where the time went

### The start-up cost

Every Glue job starts its own Spark cluster. Before Glue 2.0, that could take **8 to 10 minutes**; since Glue 2.0, AWS reduced it to **under a minute** on average, and introduced a **1-minute minimum billing** per job ([AWS News Blog](https://aws.amazon.com/blogs/aws/aws-glue-version-2-0-featuring-10x-faster-job-start-times-and-1-minute-minimum-billing-duration)). We were on Glue 2.0 or later, so start-up alone doesn't explain jobs taking up to 10 minutes. But it adds up: every job in a pipeline pays it again, both in time and in money.

This start-up cost is specific to how Glue runs Spark, not to PySpark itself. On an always-on cluster, like Databricks or EMR, jobs wouldn't pay it every time. How much that would change the picture depends on the setup, and the overhead of running small jobs on a distributed engine, which I explain below, would still be there.

### The design trade-off it led to

Because every job paid the start-up cost, we tried to have **fewer, bigger jobs** to make the most of each start. Sometimes we still had to chain several jobs with Step Functions, but the general idea was to group transformations together.

It was a reasonable trade-off, but it came at a price:

- A big job is harder to debug and to retry. If it fails near the end, you rerun everything.
- Unrelated logic ends up in the same job, just because it's cheaper to run it together.
- **The infrastructure had a say in how the pipeline was designed.**

### Spark is overkill for small data

The bigger reason for those 10-minute jobs, I think, is that **Spark is built for large, distributed workloads**. That's why the stack made sense: it scaled when we needed it, handled large volumes and fit naturally in an AWS environment. But not every job was large, and for the smaller ones, that power came at a cost.

When you run a small transformation on Spark, it still behaves like a distributed system: it splits the work across executors, coordinates them, shuffles data between them, and often writes lots of small files at the end. By default, Spark uses 200 shuffle partitions whether you have 2 GB or 2 MB (unless adaptive query execution steps in, which is on by default in recent Glue versions). For a small table, all that coordination costs more than the actual computation. That's why tuning mattered so much: `coalesce()`, `repartition()`, the number of workers… without careful tuning, a job on a small table could easily take several times longer than it needed to.

## How Snowflake handles the same work

Snowflake works very differently. Storage and compute are separated: data lives in a central storage layer, and the compute runs on **virtual warehouses**, clusters you size like a T-shirt. Tables are stored automatically in **micro-partitions**, compressed and columnar, and Snowflake keeps metadata about each one (like the min and max values of every column). When a query runs, it **skips the micro-partitions it doesn't need** and reads only the columns it uses ([Snowflake docs](https://docs.snowflake.com/en/user-guide/tables-clustering-micropartitions)). Its optimizer decides how to execute the query, and results and data can be cached.

A dbt model is just a SQL statement sent to that engine. From the developer's side, there are no executors to coordinate and no partitions to tune for a small table: the work of organizing the data is done by Snowflake.

That also changes how pipelines are designed. Each model is small and focused on one thing, and there's **no penalty for having many of them**: dbt runs independent models in parallel on the same warehouse, and the DAG decides the order. The opposite of grouping everything into a few big jobs.

To be fair, Snowflake also has a start-up of sorts: warehouses suspend when they're idle and resume automatically when a query arrives, with a **60-second minimum billing each time they start** ([Snowflake docs](https://docs.snowflake.com/en/user-guide/warehouses-overview)). The difference is that, in my experience, resuming takes seconds rather than minutes, and once the warehouse is awake, every model in the run uses it.

## Side by side

| | PySpark on Glue | dbt on Snowflake |
|---|---|---|
| Start-up | A Spark cluster per job (~1 min on Glue 2.0+) | Warehouse resumes quickly, shared by all models in a run |
| Unit of work | Job (often big, to amortize start-up) | Model (small, one per transformation) |
| Tuning | Partitions, workers, `coalesce()`… | Mostly the warehouse size |
| Data layout | You manage partitions and files | Automatic micro-partitions, with pruning |
| Minimum billing | 1 minute per job | 60 seconds each time the warehouse starts |

## So, is dbt faster?

For what we do today, many SQL transformations over data that already lives in the warehouse, yes, it feels much faster. Not because dbt is magic, but because of everything above: no cluster to start per job, an engine built for this kind of SQL workload, and a tool that fits the size of the data.

Spark still has its place. For very large volumes, heavy processing that doesn't fit well in SQL, or data that doesn't live in a warehouse, a distributed engine is often the better choice. It's not that one is better than the other: **they're built for different workloads**.

My experience with Spark is still useful, too: understanding how a distributed engine works under the hood helps me appreciate, and question, what Snowflake does for me now.

## If I wanted to measure it

I couldn't find a proper benchmark comparing the two, and my numbers are experience, not measurements. To compare them fairly, I would look at:

- **Cold vs warm runs:** the first run after a Glue cluster or a Snowflake warehouse has been idle, and the following ones.
- **Time per transformation**, not per pipeline, to separate start-up from processing.
- **Data volume:** the same transformation on small, medium and large tables.
- **Cost per run**, not just time: Glue's 1-minute minimum per job vs Snowflake's credits, with its 60-second minimum each time a warehouse resumes.

## The right question

"Which one is faster?" turned out to be the wrong question. A better one is: **which engine fits the size and shape of my data?** For the SQL transformations we run today, the answer is dbt on Snowflake. For a different workload, it might not be.
