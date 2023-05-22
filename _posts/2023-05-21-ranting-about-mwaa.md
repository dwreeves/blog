---
layout: post
title:  "Ranting about MWAA"
image: /assets/2023-05-21/airflow.png
description: A frustrated user's guide to many problems MWAA has, and some ways around them.
date:   2023-05-21
---

![Airflow](/blog/assets/2023-05-21/airflow.png)

## Introduction

Amazon Managed Workflows for Apache Airflow ("MWAA") is exactly what it sounds like: a manged Airflow service provided by AWS.

I've been using it for nearly exactly 1 year, and I have a lot of complaints about it.

In this post, I will be complaining about MWAA. This post is mainly aimed at users of MWAA or companies considering using MWAA. It serves three purposes in this regard:

1. A guide on what you can do to make your experience better.
2. A wake-up call on what we MWAA users are missing out on.
3. A call to action to demand more from AWS.

It is also aimed, to some extent, at AWS in a "get your shit together" kind of way.

Also, in this post, I will be skipping over a lot of context regarding Airflow. I will be taking for granted the reader knows about Airflow and its features.

Before we get into things though, I want to ask two things:

## Ask #1: Do not, under any circumstances, fix these problems for free

I want readers of this blog post to promise me one thing before we begin:

**Do not give Jeff Bezos / Andy Jassy free labor.**

Do not contribute code to `mwaa-local-runner` or `amazon-mwaa-examples`.

Do not open source your MWAA CloudFormation configurations.

Do not open source your AWS Lambda that updates pip dependencies.

I am going to be complaining about many things regarding the MWAA ecosystem. I'm deliberately not including code examples. I'm only providing a guide to its users of what limitations to look out for and, vaguely, how to work around them.

Some of the things I am complaining about are AWS-owned open source code examples. In theory, I suppose I could contribute to `mwaa-loca-runner` or `amazon-mwaa-examples` to fix some of these things I am complaining about.

I will not do that because I have at least a little bit of dignity. And you should not do that either. Have some dignity and don't work for free. **Don't contribute free code to fix any of these issues (unless you actually work at AWS).**

It should be on AWS to fix these tools and make them better to use, not AWS's users. Apache Airflow is already an open source project that AWS ungratefully profits from. AWS up-charges for compute to cloud users for a shitty, customer-hostile wrapper around a piece of software that is free and open source. Big tech companies build off the free labor of open source and you should not contribute to this problem in the most direct way possible. Let Amazon's shareholders foot the bill of fixing these problems.

## Ask #2: Do not blame AWS engineers for what are ultimately problems of managerial priorities

I'm not convinced that anything I'll be highlighting should be blamed on the engineers working on MWAA. Yes, using MWAA is oftentimes a bad experience. Yes, some engineer coded up the bad experience. But it is ultimately on AWS's management to make sure that the services AWS provides are good experiences, and that the tools AWS provides get adequate staffing to ensure that this is the case.

Please leave AWS engineers out of this. If you must point fingers, do so toward AWS management for not prioritizing their customers and for not having processes in place to ensure that everything is as good as it can and should be.

Behind every frustrating product decision is a managerial priority which allowed that decision to happen. It is not always the case that behind every frustrating product decision is a bad engineer, or even a bad manager.

---

With those two asks out of the way, we can now get to the meat of things.

## MWAA's development schedule is very on-and-off

It appears that MWAA is being more actively worked on these past few months. But it also feels like MWAA was more or less not worked on at all for the vast majority of 2022:

- Airflow 2.2 released in October of 2021 and was added to MWAA in late January 2022.
- Airflow 2.3 released in April of 2022 and featured _massive_ quality of life improvements for the UI; despite this huge change, it was completely skipped over.
- Airflow 2.4 came out in September 2022 and wasn't supported until January 2023.

It doesn't seem likt it should be that big of a deal to simply support a new Airflow version. It's not like they come with massive infrastructure changes! The only thing I can reasonably conclude is that MWAA had next to no staff working on it for nearly a year.

Additionally, MWAA _still_ does not have support for deferrable operators, again leading me to believe it was not developed at all in 2022; see the next section for more on that, though.

Overall, despite the more recent activity and the promising roadmap of MWAA for 2023, I would not count on MWAA to be reliably prioritized by AWS.

## MWAA still lacks a trigger instance

Deferrable operators have existed in Airflow since Airflow 2.2, which came out in October 2021. Despite this, over 18 months later, MWAA still lacks an ability to set up a triggerer instance to run these operators.

Deferrable operators are a _massive_ quality of life improvement, considering that Airflow workers often spend the majority of their time waiting for some ECS task, SQL query, or for the clock to strike 02:00 UTC. So compute allocated to synchronous workers is often wasted. Deferrable operators would be a huge cost saving. But this still does not exist in MWAA.

MWAA is reportedly working on getting triggerer instances in and they aim to have it done by end of 2023, so there is a silver lining. Still, it's unfortunate it has taken so long and continues to take so long.

## Updating pip dependencies is tedious

MWAA locks the `requirements.txt` and never changes it once deployed. There is neither an option nor even a script provided by AWS to help automatically update pip dependencies, although you can roll your own script (see further down).

The way AWS encourages you to update pip dependencies is to update the entire environment through the UI, which requires full administrative permissions and is very detached from the reality of how infrastructure-as-code deployments are managed.

Additionally and more critically, updating an Airflow environment takes 20 minutes and causes everything to restart, meaning running jobs will stall out and fail. And since there is no way to update pip dependencies without restarting everything, you'll feel the pain of this! You cannot `pip install -r requirements.txt` in a running Airflow instance without restarting everything. In theory, this should be perfectly safe to do, but again, MWAA does not really care about user friendliness.

Although you cannot get around needing to restart everything, automatically updating pip dependencies on `requirements.txt` changes is doable by setting up an AWS Lambda that, on changes to the S3 code bucket, runs a check to see whether the contents of `requirements.txt` used by the MWAA environment differs from the latest version of `requirements.txt`; if there is a change, then run an `update-environment`. Alternatively, because of the downsides of `update-environment`, you may want to forego the Lambda and instead have this be a manually triggerable job in your devops platform. At our company, we went with an AWS Lambda in Node.js, but I'm not sure if that should've actually been a Lambda or not.

## Migration to new Airflow versions is excruciatingly painful

MWAA migration cannot be done in-place, meaning that in order to deploy a new version of Airflow (even a new minor version), you are required to spin up an entirely new Airflow environment.

This is arguably one of the most customer-hostile limitations of MWAA. In any other ecosystem, you could simply run `pip install -U airflow && airflow db upgrade` and be done with it.

I personally ran into some issues with our CloudFormation setup when doing the migration, partly because our devops folks made a few manual changes that were not in CloudFormation, and partly because we did not account for having to do complete redeployments of the stack when we named things. But let's just concede for sake of argument that was an "us" problem and not an MWAA problem. Now that I've absolved MWAA of my CloudFormation mishaps, does this make the rest of the process painless? Absolutely not.

MWAA provides [Python scripts to help migrate your metadata.](https://github.com/aws-samples/amazon-mwaa-examples/tree/main/usecases/metadata-migration) (Migrating the metadata is the biggest pain in the butt in regards to having to spin up an entirely new environment.) These Python scripts are _woefully_ inadequate. You can _and will_ run into issues with them.

Let me highlight all the issues with these scripts that I've run into:

- The export script uses a 3rd party dep called `smart-open`. This is a nice library, sure, but (1) it is never actually mentioned as being a required dependency for the deployment. (2) Combine with the frustrations above about updating `requirements.txt` and you're looking at an environment update (ugh!) for your old env on top of a new deployment! Why not just use boto3?
- The import script amateurishly `try: ... except Exception as e: print(e)`s critical parts of the code, for no good reason. And these are parts of the code that are likely to fail on your first try, too! So make sure to check your green dots because they may be lying to you, and may not actually worked! (My advice: Just remove these try-excepts.)
- The import and export scripts do not migrate the `trigger` table (introduced in 2.2.0), and because the `task_instance` table has a foreign key constraint referencing the `trigger` entity, you may have a fail due to this. I did not want to deal with this, so I just replaced `trigger_id` with `null as trigger_id` in the select statement for the `task_instance` export.
- The export scripts contain a lot of `null as ...` which are vestiges of previous migrations. Compare the 2.0->2.2 migration to the 2.2->2.4 export script and you'll see exactly what I'm talking about. (To be clear, the correct pattern is you `null as ...` on the first migration, and on all subsequent migrations you are actually supposed to select the column.)
- The import scripts really mess up the log streams in two devastating and terrible ways: (1) if a single log stream already exists, the import fails completely. This is easy to work around, but very annoying that you have to do it. (2) The code barely reads in any logs at all, simply telling you `This only copies 1 MB of data. For more logs, create code to iterate the get_log_event using next_token`. What kind of maniac is only importing 1 MB of logs per log stream? Why is this the default???
- The export script for 2.2->2.x contains `AND ti.dag_id != '{dag_id}'` (this is not in 2.0, lol), where `dag_id` is the export script's DAG id, but this is incomplete. It is imperative to exclude _both_ the export _and_ import DAG ids as being excluded from the `task_instance` export. If you don't do this, then the MWAA import script will fail on a key constraint if your import doesn't work on the first go around! (Since the task instances for the import DAG will exist in the new environment's metadata store after your first run... you see the issue?).

It's clear the person who manages these scripts is probably a junior engineer. **And I'm not here to dunk on and complain about some individual junior engineer's code.** This is clearly a management issue where AWS does not care at all about its customers who use MWAA. The fact they have a junior engineer working on this in the first place, seemingly without any oversight or serious review, is the evidence of that lack of care.

But again, ultimately, this migration script thing shouldn't exist in the first place!!! MWAA should allow me to do in-place migrations without having to go through all this trouble. I shouldn't need to go through all this misery just to do what is essentially a`pip install -r requirements.txt && airflow db upgrade`.

## The `mwaa-local-runner` is imperfect

The `mwaa-local-runner` is the local runtime for MWAA. Overall, it does a decent job, and some of the issues I've been mildly annoyed by in the past (such as the hardcoded `dags/requirements.txt` path) have been fixed. Still, a few issues remain since the last time I checked:

- Fixable deprecation warnings, most notably Airflow config namespace changes.
- The password of `airflow` for the SQLA connection causes logs to censor out every instance of the word `airflow`.
- Critical deviations from the MWAA runtime that can and will screw you when you go to prod:
  - The DAGs directory should be, but is not, mounted as a read-only volume. (This will screw you depending on what tools you are doing inside your Airflow environment; `dbt` being the most notable given that it constantly writes to `target/` and `logs/`.)
  - In MWAA 2.5.1, the `pip install -r requirements.txt` when starting the webserver does not emulate MWAA's behavior of adding a local constraint file when one is missing from the requirements file.

## MWAA 2.5.1 adds a constraint for some reason (although you can cheat around it!)

This is technically documented, so I guess F me for not reading the docs carefully, but I spent a long time figuring out why my webserver instance was cryptically failing to pip install all our deps when it worked locally.

Turns out, in MWAA 2.5.1, they started enforcing that you use a constraint file in your dependencies; if you don't use one, they'll add one for you. Why? I don't know!!! The whole point of the constraint file is that it exists to serve as a stable and suggested release, but that you are also allowed to circumvent it if you want / need. It's merely a suggestion; if it was a hard requirement it would be pinned in the `setup.py`. Adding a constraint file just serves as a guardrail for the least qualified data engineers / devops engineers who don't know how to freeze deps, and pisses off people who have stable and frozen dependencies.

So, anyway, how do you cheat around it? Simple: Just add a constraint file to the requirements, but then comment it out. Dead serious. Whatever regex they're doing to enforce the constraint can be tricked into thinking there is a constraint file by adding the following to your `requirements.txt`, inclusive of the `#` to comment it out:

```
# -c https://raw.githubusercontent.com/apache/airflow/constraints-2.5.1/constraints-3.10.txt`
```

Adding this comment to our requirements allowed us to get an installation working properly.

Anyway, MWAA really, really, really should not be doing this constraint enforcement thing. This solves zero problems for anyone who isn't a novice, and only serves to create problems for people who have weird requirements. It is a massive anti-pattern as far as dependency management is concerned. And if they are going to do this (which they shouldn't!), at least provide more informative warnings + error messages in CloudWatch that this is happening. Genuinely, this ended up being a massive migration headache going from 2.2 to 2.5.

## Misc. advice on configuring MWAA

- **Be careful about over-provisioning.** An MWAA "medium" instance costs about double the amount compared to a "small" instance. If you have under 50 DAGs, you are probably perfectly fine on a `mw1.small` instance. If you do a breakdown of MWAA webserver costs vs worker costs, you'd be surprised at how much the webserver takes up as a percent of the overall costs.
- Piggybacking off the previous point: **you should probably max out your `mw1.small` workers (max 25 workers) before you consider upsizing to `mw1.medium`.** Again, webserver costs in practice end up being pretty high, so updating to `mw1.medium` just for larger workers isn't worth the price. So unless you actually need the larger webserver instance, before you scale up your workers, consider just adding more workers. It's yet another customer-unfriendly, annoying limitation of MWAA that worker size cannot be decoupled from the webserver size, but in practice this isn't _that_ big of a deal because 25 workers will usually be plenty.
- **`4,4` concurrency on mw1.small workers is far more reliable than the default `5,5` concurrency.** Despite being responsible and offloading our compute heavy tasks to ECS, we'd still hit OOM errors with Airflow workers every once in a while on `5,5` concurrency with `mw1.small` workers, but we've had not had any issues with `4,4` concurrency.
- **Your minimum worker count should be 1.** The minimum worker count setting exists to steal money from companies staffed by engineers without any business sense. You don't need 10 workers standing by idly. Airflow and Celery can spin up workers when it needs them just fine. Let autoscaling do its thing.

## Parting thoughts

I think I'm missing a few things, but I think that's plenty. I hope that if you are an MWAA user that you learned a few tricks for how to improve your own MWAA environment from this post. And maybe when you go to migrate to 2.5, you are able to fix a bug you encounter because you read about it here.

Would I use MWAA again? Honestly, I guess. Our organization does not use Kubernetes, so a nice, self managed Kubernetes + Helm installation was obviously off the table. Managing your own deployment outside of Kubernetes is a fool's errand. And now that I am very familiar with MWAA and know all the issues and workarounds, I can deal with it.

Still, it would be nice if MWAA was a better product that prioritized a smooth customer experience. It could be that one day, but it is currently not that.
