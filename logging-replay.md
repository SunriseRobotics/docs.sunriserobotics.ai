---
description: >-
  Every sunset project has robust logging built in.  Learn how to make the best
  of it.
---

# Logging / Replay

Imagine you have previously deployed a robotic system to a customer thousands of miles away from where your company is headquartered.  Then imagine that your customer who purchased such a system is having trouble localizing.

Sunset is a powerful tool to have in such a scenario because almost all topics are purely deterministic (at least those where `replace_message_with_log = False`, which is he default value) , meaning that their state  is known purely on their inputs (received messages) from other topics. &#x20;

Sunset records the data of the nondeterministic topics to a file (ie sensor inputs, this must have `replace_message_with_log = True`) and then runs that file by first populating the leaf nodes and then those leaf nodes will then send their messages to their subscribers. &#x20;

This has the very powerful effect of being able to make changes to your robot code post mortem and rerun to see if the fix is successful.  For example, in the previous example of the localization failure one can adjust filter parameters or add outlier rejection in an attempt to solve the localization issue.  You can even give the log file and your robot code to the GPT-4 advanced data analysis and it can try to solve the problem for you!

{% embed url="https://docs.google.com/presentation/d/1gEiqxhsDnoe9FK8sfDrbvJxfsWv51zIsNKaWKI-YyWA/edit?usp=sharing" %}
Presentation demonstrating ChatGPT 4's ability to use sunset logs to solve real robot issues.&#x20;
{% endembed %}
