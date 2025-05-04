---
layout: post  
title: AWS Support – An Experience That Truly Paid Off  
permalink: /2025-04-19-aws-support-experience.html  
date: 2025-04-19
---

A few weeks ago, I was working on a pretty intense project that relies heavily on **AWS Glue** as part of our data pipeline. I work at a company where we use **AWS for both development and production**, and luckily, we have an active **AWS Support plan**. Although it’s been available to us all this time, I had never used it... until now. And I have to say — it was a really positive surprise. 😌


## AWS Support Plans – A Quick Overview

Before diving into my personal experience, it’s worth mentioning that AWS offers several [support plans](https://aws.amazon.com/es/premiumsupport/plans/) tailored to different organizational needs:

- **Basic**: Free, includes access to documentation, forums, and limited AWS Trusted Advisor.
- **Developer**: Ideal for development environments, provides business hours email support from AWS engineers.
- **Business**: Commonly used in production environments. Includes 24/7 access to engineers, live chat, and phone support with fast response times (under 1 hour for critical issues).
- **Enterprise / Enterprise On-Ramp**: Offers personalized support with a Technical Account Manager (TAM), proactive planning, architectural reviews, and even more aggressive SLAs.

In our case, we use the **Enterprise plan**, which gives us access to dedicated technical resources, proactive guidance, and the ability to escalate issues when needed. 🚀


## My Experience: Trouble with AWS Glue

I was running a **Glue Job** that seemed to be working… until it didn’t. When inspecting the output, I realized that **some columns were simply missing** from the loaded data. These columns were present in the source Parquet files, but had very few values.

Glue was completely ignoring them — they were either showing up as `null` or not appearing at all unless I manually defined them in the schema. 🤯

The only workaround I kept seeing was to **explicitly include the missing column names in the schema**. But when I tried that, Glue did read the columns — yet **all the values were `null`**, even though I knew some records (just a few) had actual data in them. That wasn't good enough — I needed those **sparse but important values**.

I discussed the issue with my team, but **no one had a definitive answer** either — it was a head-scratcher for all of us. I searched on StackOverflow, forums, AWS docs, and various troubleshooting blogs. Nothing. I found other users reporting the same issue, but **no concrete solution**. After spending way too much time on this, I remembered… we have **AWS Support**!


## How the Process Went

I went to the AWS Support Center and opened a **support case**. I was offered two options:

- **Chat**, with a guaranteed response within 24 hours  
- **Live call** with technical agents

Since my project was time-sensitive, I chose the second option. Within 15 minutes, I was connected with **two support engineers** — both incredibly friendly, professional, and clearly very knowledgeable.

I explained the issue, they asked a few clarifying questions, and quickly consulted their **internal knowledge base**.

And yes — they had the solution. 😍


### The Solution: Why Are My Columns Missing?

The issue was related to how Spark (and by extension AWS Glue) handles schemas when reading Parquet files with sparse columns — columns that don’t appear in every file.

The fix was to enable **schema merging** using this configuration:

```
glueContext.sql("set spark.sql.parquet.mergeSchema=true")
````

You can check out my full answer (which helped others too!) here on [StackOverflow](https://stackoverflow.com/questions/76118661/missing-columns-in-aws-glue/79516883#79516883).



## Final Thoughts – Use AWS Support!

If your company has access to **AWS Support**, **don’t hesitate to use it**. Sometimes we feel like we have to figure everything out on our own, but having access to experienced professionals with internal documentation can make all the difference.

The support I received was **fast, friendly, and highly effective**. I’m genuinely impressed. Having access to this level of support during a critical moment made a real impact on my project.

---

Have you had a similar experience? Have you used AWS Support or another support service? I’d love to hear about it. Feel free to reach out via DM on my [GitHub profile](https://github.com/Ubikitina). 😊💬
