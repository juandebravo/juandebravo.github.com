---
layout: post
title: "Get Pull Request metrics with chat-gpt"
categories: [chatgpt, github, delivery metrics]
---

I woke up early morning today and took some minutes to read [this blog post about
engineering delivery metrics](https://leaddev.com/scaling-software-systems/primer-engineering-delivery-metrics) while grabbing my early morning coffee :coffee:.

It provides good insights about how to measure the efficiency of your
process from a Pull Request being opened until the code changes are shipped to the
production environment.

Without going into low-level details, these are the main things that should
happen during that process:

![Delivery pipeline](/gfx/posts/github-metrics/delivery_process.png)

That's something that has been running as a background thread in my mind over
the last few months, but I am always struggling to invest some time in it.

Until today! I paired with ChatGPT so I could hit two targets with one shot.

I was curious about the first part of the graph above: how long does it take for a Pull Request to get merged.

I asked:

![First question](/gfx/posts/github-metrics/first-question.png)

And GPT replied:

![First answer](/gfx/posts/github-metrics/first-answer.png)

This was a good way to start our relationship for this matter :blush: It helped me to avoid:

- checking which python library should I use.
- check the API to understand how to obtain each pull request and its status and data.

I realized my question was misleading, as it interpreted that I was interested in a specific Pull Request:

![Second question](/gfx/posts/github-metrics/second-question.png)

And GPT replied:

![Second answer](/gfx/posts/github-metrics/second-answer.png)
![Second answer](/gfx/posts/github-metrics/second-answer-2.png)

I was so impressed that I didn't even consider testing the script, I wanted more!

![Third question](/gfx/posts/github-metrics/third-question.png)

For the sake of readiness, I've put its answer in [an external gist](https://gist.github.com/juandebravo/abef866d259d610d987a9f3e08551210).

And its explanation:

![Third answer](/gfx/posts/github-metrics/third-answer-2.png)

It hadn't in mind that I was curious about the last six months of activity, and it decided to query Pull Requests from a specific date (yesterday).

So I asked again:

![Fourth question](/gfx/posts/github-metrics/fourth-question.png)

And its answer in [an external gist](https://gist.github.com/juandebravo/9c23aec131deef9f04877ba403f64d46).

It described again the solution, but to be honest I didn't read it.

At this point, I'm not sure if there's been a big gain in productivity. Perhaps the major advantage is that I didn't need to
invest some time reviewing a couple of third-party dependencies, but at the same time, I feel I'm trusting blindly what I'm being told.

Unfortunately, things starting to get a bit tedious:

![Fifth question](/gfx/posts/github-metrics/fifth-question.png)

It apologized for the confusion (so cute :cat:) and pointed to the mistake very fast (incredibly fast!)

![Fifth answer](/gfx/posts/github-metrics/fifth-answer.png)

This got me very confused and intrigued. I could understand that ChatGPT knew about `since` and `until` attributes in the `get_pulls()` method if they had existed in previous versions of the _PyGithub_ library and got deprecated for whatever reason in the last version of the library. But that was not the case, they have never existed. So I have no idea why ChatGPT decided to use `since`, it
has never existed. My two theories are that it exists in other libraries and learned from that, or it extrapolated other arguments of the REST
API and adapted its knowledge about the python wrapper. Quite fascinating IMO even though it was wrong this time.

[This is the script](https://gist.github.com/juandebravo/9ca2ea70d9d38c029809763871f9b1b8) associated with the response above. Here GPT made
a rookie mistake. We are interested in the last 6 months of data, so as soon as one Pull Request is older than that, we can stop iterating over
previous data.

I got a couple of additional errors:

![Sixth question](/gfx/posts/github-metrics/sixth-question.png)

[This script](https://gist.github.com/juandebravo/ebcd982e74a2d5da88700faa63f6f309) is interesting because it forgot to add the last line of the script:

{% highlight python %}
plt.show()
{% endhighlight %}

This reminded me to don't trust blindly the machine for now. So I read the script for the first time :laughing: and realized we were
retrieving data from the `master` branch. I can understand that because it used to be the default branch name some years ago, but [that's not the
case anymore](https://www.theserverside.com/feature/Why-GitHub-renamed-its-master-branch-to-main) :punch:.

I suggested using `main` as the default branch instead:

![Seventh question](/gfx/posts/github-metrics/seventh-question.png)

It was very polite, I was liking it a lot:

![Seventh answer](/gfx/posts/github-metrics/seventh-answer.png)

Going back to my previous problem, it was still happening!

![Eighth question](/gfx/posts/github-metrics/eigth-question.png)

And its answer and [the updated script](https://gist.github.com/juandebravo/b49db9d84e9803e780bf1741dd2e8c04), this time with the last line included!

![Eighth answer](/gfx/posts/github-metrics/eigth-answer.png)

Here we got an issue that should be easy to spot with the usual linter in your IDE:

![Ninth question](/gfx/posts/github-metrics/ninth-question.png)

And it acknowledged the mistake and provided [an updated script](https://gist.github.com/juandebravo/0b9d0c57976ec70f7de6950b4b549698).

I was getting a very ugly chart with no data, so I thought maybe we should plot "hours to merge" instead of "days to merge":

![Tenth question](/gfx/posts/github-metrics/tenth-question.png)

It explained briefly the required changes:

![Tenth answer](/gfx/posts/github-metrics/tenth-answer.png)

This is [the generated script](https://gist.github.com/juandebravo/33ba2ff6bf9c827f12d243f6e4cdce1a)

At this point, I think both of us needed a break :laughing:.

The chart was still looking pretty bad, so I did what I usually (try to) do in my day-to-day work, read the code,
understand what we are doing, add some helpful debug logs, and find the root cause of what was going on.

![Eleventh question](/gfx/posts/github-metrics/eleventh-question.png)

This was still very interesting but was not increasing my productivity anymore. This is its answer:

![Eleventh answer](/gfx/posts/github-metrics/eleventh-answer.png)

I was shocked :laughing: It indeed understood the issue (we were using datetime instead of date to create the buckets) but suddenly
it introduced numpy (`np.`) in our script.

![Twelfth question](/gfx/posts/github-metrics/twelfth-question.png)

This is its answer and [the script using numpy](https://gist.github.com/juandebravo/d15af79876e23e2c23c45af978004260).

![Twelfth question](/gfx/posts/github-metrics/twelfth-answer.png)

I tested it and it worked :tada:. In less than one hour we had a script to grab the Pull Requests from a specific repo and plot a histogram
with the number of hours it took to merge each Pull Request.

I think the job is still not ended, but it was a good start and I needed a break, so I asked it something different to have some social time together:

![Last question](/gfx/posts/github-metrics/last-question.png)

I left home at 11:00 am and the cyclist arrived at 12:30 pm. I was lucky I didn't trust it blindly.

![Bicycle](/gfx/posts/github-metrics/bicycle.png)
