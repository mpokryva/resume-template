---
layout: post
title: "How to Hire Infrastructure Engineers"
date: 2021-01-10 20:11:14 -0500
---

***Disclaimer:*** Please keep in mind that much of the advice I present in this article can and should be adjusted depending on all sorts of factors like your level of experience (and the candidate's), needs of the company, and the inherent uniqueness of people that interviews can't capture.

Infrastructure engineers design distributed systems and need to be great systems thinkers, coders, communicators, and pragmatists. I think the standard LeetCode interview process is inefficient in testing these qualities.

From my experience, it’s not too hard to get a general feel for intelligence by just talking to a person (in addition to doing a bunch of other interviews), so you’ll get a higher signal-noise ratio from having an infra-specific interview process, rather than doing standard algorithm interviews. That being said, companies often have wildly different needs for positions that sound similar, so adjust as needed.

Below, I’ll make a case for what qualities you should look (and watch out) for in engineers, and how you can assess these qualities in an interview process consisting of a system design component and a non-LeetCode coding challenge.

## On talent and traditional interviews

Before you read any further, it's important that I explain why I don't like the traditional interview process.

Excluding positions that truly use meaningfully advanced algorithms and data structures, LeetCode-style interviews are basically a proxy for IQ. There's one (possibly two) problems with this.

The first (possible) problem is that you need to believe that IQ is an effective, holistic measure for intelligence. There's a good amount of research that supports this (**note to Miki: find links**), but nonetheless, this is a controversial topic.

The second, more definite problem is that LeetCode questions are a *really* rough proxy for IQ. Actual IQ tests provide a quantitative result, but LeetCode questions provide only as much resolution as the evaluation rubric you create. Since LeetCode questions rarely map to actual job requirements, it's hard to create a sensible rubric.

It's also hard to assess what level of talent you're looking for, since you're bound to lean towards thinking that you need to hire a super talented engineer, to bolster your own ego.

In the case where you're sure you need an absolutely brilliant engineer and you're willing to reject *a lot* of people, [this](https://www.spakhm.com/p/how-to-interview-engineers) interview process *might* be a good fit. It's [highly controversial](https://news.ycombinator.com/item?id=24754748), you've been warned.

## What to look for

There are a few things that one should look for in an infrastructure engineer. System design knowledge is critical, because you don't want to hire someone that makes mistake after mistake in this area.

### System design knowledge

I think system design interviews are actually pretty good at testing this. A well-conducted system design interview can test a candidate's knowledge of the fundamentals like load balancing, caching, sharding, NoSQL/SQL, OLAP/OLTP, binary protocols like gRPC, distributed tracing, etc.

You can also dig deeper by asking questions like "What protocol is gRPC implemented on top of?" This is a good way to find competent engineers who are able to peel back abstractions when needed.

In a similar vein, constantly asking candidates to explain their answers is a good way to tell apart those who’ve just watched a bunch of system design interviews on YouTube from those who actually understand what they're talking about.

For example, if a candidate says that using an auto-incrementing primary key as an external identifier for a resource is problematic, ask them why that is. On the other hand, if they suggest to use it as an external identifier, ask them to figure out why that can be problematic.

A more generic example of this case is when a candidate confidently suggests `$buzzword` as a solution. Ask them why they suggested `$buzzword`. If they don't have an explanation, that's not a great sign.

### Ability to write clean code

I'm sure you've heard some version of [Robert C. Martin's](https://en.wikipedia.org/wiki/Robert_C._Martin) quote:

> “Indeed, the ratio of time spent reading versus writing is well over 10 to 1. We are constantly reading old code as part of the effort to write new code. ...[Therefore,] making it easy to read makes it easier to write.”

I don't think this is a wrong or particularly controversial opinion. Unfortunately, the interview processes of most, if not all, companies I've interviewed with had little to no focus on testing the readability of my code.

These are two good ways to see if a candidate can write good "real" code:

1. Ask the candidate for an open-source code sample, if they have one.
2. Give them a take-home or live API design challenge that they can implement, perhaps mocking datastores or whatever other details aren’t reasonable to expect from a coding challenge. 


In assessing the above, you can look for a few things:

- **Implementation of clean abstractions** between application layers via design patterns.
Good naming of variables, functions, classes, etc. Different people have different opinions on this, so I won’t dwell.
- **Good documentation** is a good way to see if people’s technical communication skills are up-to-par. It’s especially important to be able to write good documentation if you’re going to be writing a lot of systems that communicate with each other, and where most of the problems occur at integration points.
- **Unit tests** that catch edge cases and mock things like the filesystem/network cleanly.

### Curiosity 

Asking candidates to tell you about one or two interesting things they’ve learned, or books they’ve read is a nice way to see if they’re just doing what everyone else is doing, or whether they’re genuinely curious and are learning new things.

Bonus personal [example](https://www.usenix.org/legacy/event/lisa07/tech/full_papers/hamilton/hamilton_html/index.html) - an awesome paper about building web services by [James Hamilton](https://mvdirona.com/jrh/work/) I read a little while ago.

### Humility

Designing distributed systems and interfaces is challenging and often involves a lot of mistakes. It’s important to look for people that understand this, and here are a few specifics to look for:

- **Engineers that think in a “failure-first” mindset.** Rather than designing the “perfect system,” it’s important to assume failure will happen (and more often, as you scale) and apply some stability patterns like load balancing, redundancy, timeouts, retries, and circuit breakers.
- **Engineers that are willing to change their minds** during the system design interview if you suggest better solutions.
- **Engineers that design coherent, intuitive, and evolvable APIs.**


### Track record of “above and beyond” achievements at past jobs or projects

This point is directed more towards early-stage startups that, like it or not, need their employees to go above and beyond. 

If the following statements are true then you shouldn’t expect them to have a wildly different level of impact at this company than they did at their last one.

1. Your company isn’t doing anything that they seem genuinely passionate about (it’s hard to be honest about this).
2. They don’t have a high potential financial upside from the company succeeding.

Ask yourself (genuinely), “if they haven’t gone above and beyond at previous jobs, why would they do it here?” If you can’t find a good answer then you might not want to risk hiring them. 

Note that the answer to the above question doesn't have to be clear cut. Maybe you just have a gut feeling from talking to them that they’ll be an amazing employee, and maybe you should trust your gut.

## Red flags to look for

**Jumping to solutions too quickly.** Distributed systems are big and complex, and making a lot of mistakes needlessly is expensive.

**Not jumping to solutions quickly enough.** Distributed systems are big and complex, and you will make mistakes along the way, so at some point, decisions have to be made.
Obviously, these last two points are a bit tongue-in-cheek, but this is an extremely important balance to look for in an infra engineer.

**"Ivory-tower architects" or [Architecture Astronauts.](https://www.joelonsoftware.com/2001/04/21/dont-let-architecture-astronauts-scare-you/)** The first term is a reference from the book [Release It!](https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213) by Michael Nygard (which I highly recommend) which describes a type of engineer that creates ideals not rooted in reality and then forces those ideals upon all other engineers at their company. The second describes a type of engineer that finds patterns in problems (as all good engineers should), but then over-generalizes these patterns into useless abstractions.

While these terms aren't synonymous, they similarly describe a type of engineer that often works too slowly, over-engineers solutions, and has very strong opinions about all tech that things that ultimately don't matter too much (**idk about this last one**).

I find that these engineers are more common among infrastructure engineers and API designers, whose work is often highly technical, hard to reverse, and stays around for a long time, so there's more potential for over-engineering. I’ve experienced this firsthand multiple times (and I’ve done it myself!), and it ain’t fun:(

A red flag for this is when candidates use a lot of buzzwords and talk about “best practices” with no mention of their downsides or alternatives when suggesting solutions to design problems.

**When something just feels "off."** From my experience, when people feel that something's "off," they're usually right. I once shadowed an interview where the candidate had great experience and technical competence, but for reason, I was just *annoyed* by them. I thought more about why that was, and I realized that they were constantly giving their take on everything, even when neither I or the main interviewer asked for it. I wrote this down in my feedback and gave them a "No Hire," which I felt bad about since I realized this was a relatively strange reason to not hire someone. A few days later, to my surprise, I found out that several other people expressed the same thing, and we ended up not hiring this person.


### **`$conclusion`**