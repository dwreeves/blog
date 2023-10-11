---
layout: post
title:  "Rich-Click 1.7.0 Released"
image: /assets/2023-10-11/rich-click-logo-opaque.png
description: Release notes on Rich-Click 1.7.0
date:   2023-10-11
---

![Rich-Click](/blog/assets/2023-10-11/rich-click-example.svg)

I am a newly onboarded co-maintainer of **Rich-Click**, and I am happy to announce that we are releasing **Rich-Click 1.7.0** after a 10 month release hiatus, and it was worth the wait.

> Major thanks to **Kory Taborn** ([@BrutalSimplicity](https://github.com/BrutalSimplicity)) for his massive contributions to the 1.7.0 release, **Phil Ewels** ([@ewels](https://github.com/ewels)) for authoring and maintaining **Rich-Click**, and to all the contributors to both [Rich](https://github.com/Textualize/rich/graphs/contributors) and [Click](https://github.com/pallets/click/graphs/contributors).

The new release does the following:

- **Full static typing support.** We are at full parity with Click 8.x's static typing support. No more Mypy yelling at you!
- **Deprecating Click 7.x support.** Click 8.x makes a handful of great improvements to Click's internals. Removing Click 7.x support will open up a handful of new opportunities for Rich-Click down the line. That said, Click 7.x is still almost fully supported in Rich-Click 1.7.0.
- **A plethora of improvements to achieve Click feature parity**, such as patching the `click.MultiCommand` and `click.CommandCollection` objects (which are also now part of `rich_click.cli.patch()`), handling all ways of defining commands/groups that are valid as of Click 8.1, handling callback invocations exactly the same way Click does (across both Click 7 and 8), and more.
  - If that is too in-the-weeds, I can put it in simpler terms: **Rich-Click** behaves exactly like Click more than ever, down to the most painstaking, minute detail.
- New style options:
  - `click.rich_click.STYLE_COMMAND`
  - `click.rich_click.WIDTH`
  - `click.rich_click.STYLE_ERRORS_SUGGESTION_COMMAND`
- `@rich_config(help_config=RichHelpConfiguration(...))` decorator, which lets you customize in a scoped interface (previously options could only be changed by modifying globals in the `click.rich_click` module).

If you have ever tried **Rich-Click** in the past and you ran into a minor snag (such as with static type-checking failures, or any other minor bugs or limitations), I implore you to give **Rich-Click** another shot. The latest version of **Rich-Click** has been thoroughly tested and revamped.

If you have never used **Rich-Click** but you have used Click, there is no better time than now to start using **Rich-Click**.

If you have never used either Click or **Rich-Click**, perhaps I can convince you to step foot into the most popular third-party CLI ecosystem in Python? 😊

# Why I Love CLIs

It took me a little bit to pick up on using Command Line Interfaces (CLIs). The more I used them, the more it "clicked" for me (pun slightly intended). When you first get into Python scripting, you end up writing a lot of code that looks like this:

```python
import foo

def main():
    ...

if __name__ == "__main__":
    main()
```

But it turns out that defining an **_explicit interface_** in front of your code is a much better way to interact with your code. It's useful at all scales: tiny single-file scripts, small applications in Docker containers, and large frameworks all benefit from CLIs. And you don't even need different tools for all of these scales: the popular CLI framework [Click](https://click.palletsprojects.com/) is a great choice for both tiny and massive codebases.

```python
import foo
import rich_click as click

@click.command()
def main():
    ...

if __name__ == "__main__":
    main()
```

# Why I Love Click and Rich-Click

![Click](/blog/assets/2023-10-11/click.png)

Click has always been my favorite Python CLI tool. Despite the numerous alternatives that boast fancier, DRYer abstractions and smooth developer experiences for simple CLIs, Click has always been the go-to, bread-and-butter CLI tool for big and popular frameworks spanning across many domains, such as:

- Flask (of course)
- dbt-core
- Celery
- Dagster
- Bokeh
- Catboost
- DALLE2-pytorch
- Airbyte's Octavia CLI
- Apache Airflow's Breeze CLI

Why does Click remain so popular despite the existence of fancier alternatives?

- It is battle-tested.
- It is very composable.
- It is fairly explicit.
- The abstractions are well-designed such that you can do basically anything with it; very few custom features require jumping through hoops to implement.

**Rich-Click** is an attempt to make Click better by integrating it with [Rich](https://github.com/Textualize/rich) to render prettier help text:

![Rich-Click](/blog/assets/2023-10-11/rich-click-example.svg)

**Rich-Click** was designed to be a drop-in replacement for Click. It does not force you into a new API. All it asks of you is to find each line of code where you do this:

```python
import click
```

And replace it with this:

```python
import rich_click as click
```

And that's it!

I really enjoy tools like **Rich-Click** that piggyback on perfectly fine, existing APIs. Tools that shim existing APIs respect your time: both the time you committed to learning Click, and the time you would have counterfactually had to spend learning a new tool. These tools know you have a life and responsibilities outside of learning the hot new framework of the month.

# Making Rich-Click Resilient

![Rich-Click](/blog/assets/2023-10-11/rich-click-logo.png)

**Rich-Click** has always been a fine replacement for Click, but it could hardly hold a candle to Click itself in terms of reliability and robustness. This is not a dig on **Rich-Click**, rather it is a commendation of the incredible amount of effort and ingenuity that has gone into both designing and maintaining the Click library.

Our objective with 1.7.0 was to make sure that **Rich-Click** could hold a candle to Click's own resilience. This meant:

- Adding unit-tests
- Adding `--strict` static type checks.
- Integration testing against existing Click and **Rich-Click** CLIs in the wild.
    - (To my knowledge, Apache Airflow's Breeze CLI is the most thorough **Rich-Click** implementation out there, so I made sure the 1.7.0 release worked perfectly with it).
- Diligently fixing bugs; never saying "no" to fixing something no matter how small or hard it was to fix.

And I do believe we have achieved our goal.

# What's In Store for Rich-Click In The Future

I'd rather under-promise and over-deliver than the reverse. I am very confident that within the next 6 to 12 months, we will be able to achieve all of the following:

- **Real documentation:** Right now the README serves as the single source of documentation for Rich-Click. We hope to soon expand the documentation so that every single feature in the API (and there are quite a few!) is well-documented.
- **More minor formatting options:** There is a lot of demand for different ways to customize **Rich-Click** help text. While we will not always agree on what a default behavior should be, we all agree that **Rich-Click** should be a highly customizable experience, and we want to support as many configuration options as we reasonably can.
- **Better integration into `click.HelpFormatter` object:** This would be a restructuring of rich-click's internals that would not have direct use to typical users, but it could enable some interesting customization functionality down the line.
- **Support for arbitrary Click extensions:** **Rich-Click** isn't the only Click extension I like; I am also a fan of [`click-didyoumean`](https://github.com/click-contrib/click-didyoumean). It's not clear right now whether we will natively integrate extension functionality, or do some ✨magic✨ so that `@rich_click.command(cls=DYMCommand)` will render Rich marked-up help text. In either case, we share your desire to add more extension functionality to **Rich-Click**.

And once we get all of that done, who knows what comes next. ☺️
