---
title: A/B Testing Overview
author: Emily Robinson
date: '2019-11-18'
slug: a-b-testing-overview
categories: []
tags: []
keywords:
  - tech
---

## What is A/B Testing?

A/B testing is the process of running an experiment to test the effect of introducing a change on your website, email, or other online communication. Visitors to the site or recipients of an email are randomly assigned to either the control or treatment group. The random assignment ensures that there’s no pre-existing difference between groups that could explain any differences we see.

## Why is A/B testing necessary?

Why can’t we make a change and watch the metrics? Let’s say normally 5% of unregistered visitors to the homepage end up registering. We change the homepage module and lo and behold, the registration rate in the week afterwards is 7%!

But there are multiple problems with this approach:

There could be seasonal effects. What if we ran the test during a holiday (or promotional week, or the week that everyone in the world received $1000 from the sky), which is actually what caused the increase rather than our experiment.The changes may be smaller than natural variation. It’s unlikely our registration rate per day is always exactly 5.00% - more likely, we could find it ranges between 4.76% and 5.45%. Let’s say we introduce the change and it actually increases registration rate by 5%: if registration rate would have been 5% on a day, it’s now 5.25%. This could be a meaningful change in our metric, but we wouldn’t detect it on a time series graph - it’s small enough to fall within the normal daily fluctuation.A/B testing controls for this by giving the counterfactual: in this week, with identical people (if we randomize correctly), how did the old perform against the new?

Why do we need statistics to do this? If we have a higher registration rate in B vs. A, why can’t we just say B is better and call it a day?

In experiments, we’re getting just a sample of the population. While we may be testing on all people that visit a page on that day, we can’t look into the future and test on people who will visit in two weeks. And a sample means we have variation.

For example, let’s say we flipped two coins. One turns up heads 27 times out of 50 and one 26 times out of 50.  Would you conclude the second coin is more likely than the first to turn up heads?

No! You understand there’s randomness at play. But what about if we got heads 34 times out 50 for that coin? How can we quantify where we draw the line that says, “Ok, now I’m willing to believe the coins have different probabilities of turning up heads?” Statistics!

## What are the components to an A/B test?

The null hypothesis is the hypothesis that there’s no difference in the metric between the two (or three/four/etc.) groups. We’re trying to disprove it - we’re running these tests because we think we’re going to make a difference! Just like in a jury trial where someone is innocent until proven guilty, we consider both variants the same until proven different.

When we do a statistics test, we’re deciding whether we’re going to reject the null hypothesis or fail to reject it. Rejecting it means, “given our data, I’m pretty confident there’s a difference between our groups.” Failing to reject it means, “given our data, I’m not able to say there’s a difference between the groups.”

We do this by comparing the difference we found to the typical range of differences we’d find between two groups if there is no real difference. We’re trying to answer the question: if there wasn’t a difference between these two groups, how likely is it we’d get the result we had in our experiment?

The p-value answers that question: it’s the probability that, if there is no effect, we would see a difference at least this extreme. A common threshold to decide when we can reject the null hypothesis is .05.

The p-value threshold affects the false positive rate. If we say, “We’ll reject the null hypothesis for any test where p < .05”, we’re accepting that 5% of the time, we’ll be saying there’s a difference when there is none! But a p-value of .04 does not mean that this specific result has a 4% chance of being a false positive. To understand more about why, check out [this thread](https://twitter.com/methodsmanmd/status/997482408973922305?s=20).

## Planning an A/B Test

“To consult the statistician after an experiment is finished is often merely to ask [them] to conduct a post mortem examination. [They] can perhaps say what the experiment died of.” - R. A. Fisher

### What’s an A/A test? When should I run one? And how long do you run it for?

An A/A test is when you test two identical versions of a page against each other, usually to check your experimentation system is working correctly. Whenever we want to start experimenting on a new part of the site (e.g. on Campus), we run an A/A test.  
In an A/A test, you check both that the health metrics look good and that you aren’t seeing significant differences in the conversion metrics. The first is something you could find out in an A/B test - for example, if you have bucketing skew or many of your experiment starts don’t have cookies. But you would not be able to see problems with the conversion metrics from an A/B test since you might expect different rates in the control and treatment because of your change! Even for the health metrics, it’s nice to find problems before some people are exposed to your treatment  - otherwise, you’ll have some pollution where some people in your fixed experiment will have seen the treatment before. 

On the engineering side of the A/A test, you want to make it as close as possible (ideally exactly the same as) the real test. So, for example, all of the conditions should be exactly the same as they would be for the full experiment, and both the if and else blocks (if written that way) should just return the control logic for now. (This could mean returning the same value, returning a no-op, or dispatching the same action depending on how it's implemented.)

This also means that when you go to implement the full experiment, all that should be necessary is changing the else block (if implemented this way) to return the treatment logic. This way, you can ensure that the A/A test is exactly representative of the way the full experiment would be run.

Determining how long to run an A/A test is tricky. “As long as possible” is the real answer - the longer you run it, the smaller the problems you can detect. For example, let’s say you ran it for a week, which was powered to detect a 5% change in course starts. If something is wrong in the experimentation system but it only messes up metrics by 2%, you wouldn’t see that. In practice, you want to run it at least as long as you’ll be running your typical tests. Generally I would recommend at least two weeks. 

Note: especially when looking at multiple metrics, it's always possible that by chance, you'll see a significant p-value even if everything in your experiment is working properly. To minimize this possibility, make sure you're following best practices by only checking on the day you set as the end of the experiment (see the peeking section below). At the end of the experiment, the lower the p-value, the more likely something is wrong. If your p-value is between .01 and .1, let's talk; if it's < .0001, something is definitely wrong. 

### Power

Power is the probability that, if there’s an effect of X size, we would detect it (e.g. p < .05). This lets us know what our false negative rate will be. For example, if we have 80% power to detect a 10% change that means, if there actually is a 10% increase, 20% of the time p > .05 and we don’t detect the change!

Power increases with three things: our sample size (the number of visitors per day), the baseline of the metric (e.g. a 30% click rate vs. 10%), and the effect size (how big a difference). If it’s a proportion metric (e.g. click rate or percentage of people registering), you can calculate how long an experiment would take to run with http://www.experimentcalculator.com/. Enter the number of people who would be affected each day in the first box, the conversion rate in the third, change "how confident do you want to be of this" to "90" (as we use p values of .1 rather than .05), change the fourth box to the effect you think your experiment will, and look at the number of days at the bottom. This is how many days your experiment would need to run for to have 80% power. 

We’re concerned about power because it lets us know how long we’ll need to run an experiment for or even whether it’s worth running one in the first place. For example, let’s say we have a brilliant idea to increase registrations on the landing page of Introduction to Python for Data Science course. We have about 7500 visitors per day. Let’s say they 1500 of those are already registered. Of the other 6,000, generally 5% of them register. If we wanted to detect a 5% increase in this metric (to 5.25%), it would take 41 days, which we decide is too long to run an experiment. We would either need to increase the size of the change we hope to detect (e.g. a 10% increase would take 11 days) or pick another metric with a higher baseline, such as course starts, to target.

Picking a reasonable effect size in the beginning will be difficult. Once we’ve been A/B testing for years, we’ll be able to use the outcome of similar past experiments to make an informed guess. We’ll also have picked off the lowest hanging fruit (with the biggest effects) and be working on smaller optimizations. Until then, we can use product intuition, user studies, and the magnitude of our change to make an estimate.

### New Metrics

What about the case where we’re adding something that wasn’t there before? For example, let’s say we want to add a link to the business page to an email. The current rate of that metric is 0, so we can’t use a traditional power analysis to calculate how long we should a new test.

There are a couple factors at play here. First is running the test long enough to capture the natural variation. For example, maybe we get a high click rate on the weekdays but not the weekend. In general, we want to stick to at least a week ,but the longer we run it, the more confident we can be.

We also want to be cautious about the novelty effect. For example, if we add a button to a page, then for everyone who visits that day it will be the first time they see it. Perhaps we’ll get a click-rate of 40%, but once people have seen it a few times or clicked it once, they won’t click it. We want to give people time to adjust.  

Finally, we can use a technique called bootstrapping to estimate the likely range of the true value for a metric.

### Prioritizing experiments

We have dozens of ideas for experiments. How do we decide which ones we should try first?

Opportunity sizing is a way to rank experiment ideas by their potential impact on our key metric. Experiments targeting the same metric are compared to each other. For example, let’s say we have 20 experiment ideas targeting registrations. For each idea, we will calculate how many visitors would see the change and what their current registration rate. We then ask, “If each experiment increased registrations by 10%, how many more total registrations per day would we get?” The formula here is (# of daily visitors) * (Current registration rate) * (Percentage change in registration) = number of new daily registrations.  

Therefore, if experiment A has 10,000 daily visitors and a registration rate of 5% (500 users per day), a 10% increase would bring in 50 more users per day. On the other hand, experiment B has 30,000 daily visitors but a registration rate of only 1% (300) and so a 10% increase would only bring in 30 more visitors per day.

Of course, this assumes each experiment would change the metric by the same amount. If we have strong reasons to believe an experiment is likely to make a bigger change, we can adjust for that. But we should not be too confident in our ability to predict likelihood of experiment success or size of impact.

### Coordinating with other teams

As the growth team, we’ll be working on projects across the company. Sometimes we may want to run a test on a page at the same time as the main-app team. How do we handle that?

The first question we need to ask is: can both changes be live at the same time? For example, if our test is to change the case study banner on the signed-out homepage to user stories and theirs is to change it to pictures of instructors, both of those cannot be launched. In this case, we should either combine them into one experiment with multiple variants, postpone one, or eliminate one test. If the tests have different start dates by a few days, we can delay one of them to combine with the other. If one is going to be ready more than 5 days before the other, we should launch it; there are many reasons why the other might be delayed. We also shouldn’t be running tests longer than two weeks generally, so it won’t delay the second experiment too much.

If they’re on different parts of the page, we should see if we can sequence them. Especially if the overlap would only be a few days, by delaying the second one we’ll know how it performs with the first. While we have the ability to ensure that visitors are only ever in one experiment, what if both experiments are successful? We won’t know how the combination performs. It will also double the time each experiment needs to run. Unless one of them does unexpectedly well or poorly, we won’t have saved any time.

To facilitate coordination, we should have a central place where we outline what tests we will be running and which ones are currently live.

If the experiments are not running on the same page, we should not worry about interactions. Interactions matter when they lead us to make different decisions than we would have otherwise. Most experiments don’t even make a difference. We already have a limited amount of traffic to our site and dividing it between two experiments will significantly decrease our velocity.

### Overall Criterion and Monitoring Metric

Generally, we should be targeting a change to one metric per experiment. Most of the time, this should be subscriptions or B2B SQLs. For changes targeting users who are already subscribed, we’ll aim to find metrics that predict them renewing.

We should form a hypothesis around this metric to explain why we expect it to change. For example, for a test that adds the certain copy to the top of the signed out homepage, we might write, “We expect this message to increase subscriptions by more than 10% because social proof has been shown to be one of the most effective ways to increase conversion.”

In addition to our target metric, we’ll want another set of monitoring metrics. This helps make sure we aren’t having adverse impact on other parts of the business. For example, in an experiment targeting an increase in B2B leads, we’ll want to monitor individual registrations to make sure we don’t tank them.

We can use the guidelines from a paper (that unfortunately I can't find the link to, but the advice doesn't need much justification) to make decisions using multiple metrics: 
  1. If all key metrics are flat (not stat-sig) or positive (stat-sig), when at least one positive, then ship the change.
  2. If all key metrics are flat or negative, with at least one negative, then don’t ship the change. 
  3. If all key metrics are flat, then don’t ship the change and considering either increasing the experiment power, failing fast, or pivoting.
  4. If some key metrics are positive and some key metrics are negative, then decide given the tradeoffs. When you have accumulated enough of these decisions, you may be able to assign weights.

For number 4, deciding the trade-offs initially will need to be down in conjunction with the other teams our work has affected and our own OKRs.

### How can we look at retention?

First, don't worry that by changing retention you'll be messing up the accuracy of metrics. For example, you might wonder if it's a problem that while you'll get 100 new people on the first day, if treatment does increase retention you'll get 90 returning people on day 2 in treatment and only 80 returning in control, plus 50 new people in each. It's not because we track people, not visits. So when new people come in, we’re still randomly assigning them. Therefore, while you may have 240 visits over the 2 days for the control (100 day 1 + 90 returning day 2 + 50 new day 2) vs. 230 in the control, you'll still have 150 in each.  

But retention metrics are tricky to monitor. If you're looking at monthly retention, obviously you can only measure that a month after someone enters an experiment! You have two options for how to run experiments. One is to run for say two weeks,  turn treatment completely off, and then check in a month and look at each person’s metric one month from when they started. But if the treatment is something ongoing (like a daily reminder email), the treatment group doesn't actually reflect what would happen if you rolled it fully out, because they stopped getting the treatment after two weeks, whereas if it fully launched they would have been getting it the full month afterward as well. 

The second option is to just keep running the experiment for a month and a half. But then we’ll always have people entering who only have one day of data, so it's unclear what to do with those people. 

Probably the best option is to stop putting in new people but keep the treatment change running for the people who entered the experiment. But then you can’t run conflicting experiments for the next month. 

### How many variants should we run?

The more variants we run, the longer the test will need to run. It’s important to think about how we can narrow down the number of variants before the test.

As a rule of thumb, we should limit the number of groups to four (A/B/C/D). Deciding whether we can do 2, 3, or 4 groups will depend on our power analysis - if we have a smaller sample or low rate of the key metric, we should favor doing only an A/B test.

### What size should each variant be?

Some people think it’s “safer” to run a risky variant with a smaller percentage (e.g. 10%) since if there’s a problem it will affect fewer people. But this doesn’t really work because you’ll need to run the experiment longer to detect an effect. It would also be harder to detect a problem using our real-time monitoring systems (like the website going down or pageload time increasing) if only a small proportion of users are affected.

The exception is email tests. Unlike online tests where we get a (relatively) steady stream of visitors, have real-time monitoring, and won’t reach more than 200k+ people for weeks, if ever, one email can be sent out to more than 1 million people. If there’s an email change we deem especially risky (e.g. might lead to make a lot of people angry and unsubscribing or reporting it as spam), we can run it on a smaller percentage of users. The downside of not being able to detect as small effects as a 50/50 test remain the same, so we should only do a smaller test in rare cases.

For the same reason, we should not run a test with 95% in the new experience and 5% in the old. Even if we’re pretty confident it will be better, we’re running the test to verify that! It will be slower to detect an effect, positive or negative, with an uneven split, and we may never be able to detect the smaller effects that we could with a 50/50 split.  

To reach conclusions as fast as possible, we should run tests with equal numbers in each group (e.g. 50/50 for an A/B or 34/33/33 for A/B/C).

### Launch on Neutral?

While we’d love if every test resulted in a 500% increase in our key metric, the reality is that most experiments don’t move the needle, or at least not strongly enough for us to detect it. In 2009 when Google ran over 12,000 experiments, only 10% lead to business changes. So what should we do if we don’t see a change in our metric?

The first is to remember that a no change is always a “likely no change bigger than x size,” not “there’s definitely no difference at all between the groups.” For example, let’s say we ran a test where we calculated it would take 10 days to get 80% power to detect a 5% increase in registration rate. Unfortunately, after 10 days p = .45. But it could be that there was a 3% decrease in registrations, which we don’t have enough data to detect.

That’s the danger of releasing on neutral - we might have introduced a negative change to the site that, while too small to detect in our change, still has a meaningful impact in the long-term.

So how do you decide what to do? The first step is to look at the other metrics. If other metrics you care about have been impacted negatively, you’ll probably want to rollback. If not, this is where your product intuition and other data can come in. Is this a change users have been asking for? Does it set the foundation for future changes we want to make?

## Monitoring an A/B Test

### Bucketing skew

Bucketing skew, also known as sample ratio mismatch, occurs when we have a different number of people in each group than we expected. For example, imagine we’re running a 50/50 test, but we find that 48% of users are in A and 52% are in B, and a proportion test finds that is a significant difference (p < .05). Because treatment assignment is independent of the treatment, the split of users between the variants should be what we defined. Otherwise, we have a bucketing skew. [This paper](https://exp-platform.com/Documents/2019_KDDFabijanGupchupFuptaOmhoverVermeerDmitriev.pdf) discusses some debugging methods and also why even a small skew can have a big impact.

Another type of skew would be if we had the same number of people within each group but they were had different pre-existing characteristics. For example, imagine in A we had almost exclusively monthly subscriptions and in B almost exclusively yearly subscribers. Any differences between A and B would likely to just be an artifact of the population difference.

This would be a failure of random assignment. If the experiment is set up correctly, population characteristics will be the same (or very close) across groups. We can set up a check for a few important characteristics that they’re the same.

### Peeking

First, a disclaimer:

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">What I tell people: Don&#39;t peek at your A/B test early, it&#39;s meaningless.<br><br>What I do: Come on, B! It&#39;s been twenty minutes, you can beat A!</p>&mdash; David Robinson (@drob) <a href="https://twitter.com/drob/status/704425627479191552?ref_src=twsrc%5Etfw">February 29, 2016</a></blockquote>


Scenario: our test is up and running. We check in on it every day to make sure it’s doing okay. On Day 4, we see registrations have gone up in group B vs. A, with a p-value of .045! We were originally planning to run the test for a week, but now we can conclude it and call it a day, right?

Wrong. What we’re doing here is called peeking and it increases the false positive rate. Dave wrote a [blog post](http://varianceexplained.org/r/bayesian-ab-testing/) where he shows this problem by simulating thousands of experiments where there was no difference between the two groups. If you check over day over 20 days, more than 20% of the experiments will drop below p = .05! If we don’t guard against peeking, we’re going to think we’re having a lot more successful experiments than we really do. That's why we always set an end-date for our experiments. 

### Real-time metrics

The idea of seeing A/B tests in real-time can seem exciting or even necessary. We want to make decisions as quickly as possible, right? In actuality, using real-time metrics of A/B tests to make decisions is a bad idea.

The first problem is we almost definitely won’t have enough data after a few hours to have power to detect changes. The exception would be if we have a bug that breaks a button or keeps the page from loading on certain browsers. But in those cases we should not rely on our A/B testing system to catch these problems; we should use the engineering systems we already have set up. The second problem is being able to see new results multiple times a day instead of just once exacerbates the peeking problem covered above.

If you want to learn more, check out [this blog post](http://mcfunley.com/whom-the-gods-would-destroy-they-first-give-real-time-analytics).

### Multi-armed bandits (MAB)

MABs are an alternative to A/B tests. Instead of setting up an experiment that randomly assigns people at the same rate over a set period of time, MABs make decisions on what proportion to assign to each group based on past performance. If a group has been performing well on your key metric, you will assign more of your new visitors to that group. You’ll alway assign some people to every group, just in case the initial well-performing group isn’t actually the best one.

The advantage of MABs is that you optimize for your metric in the test period. The downside is that you take longer to reach statistical significance. That means that while you may have gotten more subscriptions in your test period than you would have with a traditional A/B test, you won’t be able to leverage that knowledge in the next test because you won’t know which variant performed the best.

MABs also rest of the assumption that the conversion rate is the same at all periods. This is big assumption that does not hold for the vast majority of websites. The likelihood that someone subscribes or registers is different on a Tuesday at 3 am then Wednesday at 5 pm. There are some computational methods to tackle this problem, but I recommend we lay the foundation first to run A/B Testing and consider MABs at a later stage.


### I’ve fallen in love with A/B testing and want to learn more. Where can I go?

Excellent question! There are so many resources out there for A/B testing, especially when you expand it to include resources on traditional experimentation as well:
- [How to](http://datadriven.club/) start experimentation at your company and do opportunity sizing
- How you can still do [big changes](http://iterative.club/) with A/B testing. 
- More on [how to](https://causalinference.club/) start experimenting. This is by the same author as the previous two and includes his recent experience helping MailChimp do this. It's also more focused about how A/B testing requires changes to engineering mindset.  
- [Review](https://www.widerfunnel.com/3-mistakes-invalidate-ab-test-results/) of basic but common mistakes in A/B Testing
- [Meta-analysis](https://www.qubit.com/wp-content/uploads/2017/12/qubit-research-meta-analysis.pdf) of online experiments. The main finding is that scarcity, social proof, and urgency experiments do well, while simple UI changes are often ineffective.
- Evan Miller’s [website](https://www.evanmiller.org/) has some great resources, including experiment calculators and articles about the [peeking problem](https://www.evanmiller.org/how-not-to-run-an-ab-test.html) and [why](https://www.evanmiller.org/lazy-assignment-and-ab-testing.html) you should include only visitors who see your change in your analysis.   
- [How Booking.com democratized online experimentation](https://arxiv.org/pdf/1710.08217.pdf). This is a quick read (7 pages, no math or stats) and is a nice view into a very mature experimentation system.  
- [Four principles of experimentation](https://medium.com/airbnb-engineering/4-principles-for-making-experimentation-count-7a5f1a5268a) from AirBnb. 




