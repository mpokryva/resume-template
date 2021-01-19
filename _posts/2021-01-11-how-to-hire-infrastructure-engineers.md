---
layout: post
title: "How to Hire Infrastructure Engineers"
date: 2021-01-10 20:11:14 -0500
---

***Disclaimer:*** Please keep in mind that much of the advice I present in this article can and should be adjusted depending on all sorts of factors like your level of experience (and the candidate's), needs of the company, and the inherent uniqueness of people that interviews can't capture.

I'm also aware that there are [plenty of companies](https://github.com/poteto/hiring-without-whiteboards) that don't do algorithm/data structures (DS) style interviews, and have processes similar to what I describe here. I figured it might still be an interesting read for some people, and also I'm bored in quarantine and wanted to get back into writing. 🙂

# On traditional interviews

Infrastructure engineers design distributed systems and need to be great systems thinkers, coders, communicators, and pragmatists. I think the standard algorithm/data structures (DS) interview process is inefficient in testing these qualities.

From my experience, it’s not too hard to get a general feel for intelligence by just talking to a person (in addition to doing a bunch of other interviews), so you’ll get a higher signal-noise ratio from having an infra-specific interview process, rather than doing algorithm/DS interviews. That being said, companies often have wildly different needs for positions that sound similar, so adjust as needed.

Below, I’ll make a case for what you should look for (and what you should watch out for) in engineers, and how these qualities can be assessed in an interview process consisting of a system design interview with an optional coding challenge (more details on that below).


*Note*: I’ve also had a “hybrid” interview once that I liked, where I was asked to implement a simple algorithm and talk about optimizing and scaling it in the real world, similar to a system design interview. I won't go into that here.

# What to look for

### System design knowledge
Load balancing, caching, sharding, NoSQL/SQL, OLAP/OLTP, binary protocols like gRPC, distributed tracing, etc. 

If you get far enough you can see how much a candidate knows about some of these (i.e: What protocol is gRPC is implemented on top of?). At some point this gets to be trivia, but generally, asking these questions is a good way to find competent engineers who are able to peel back abstractions when needed.

### Ability to reason about architecture design decisions
This signal can be measured well from system design interviews. In these interviews, it’s easy to tell apart people who’ve just watched a bunch of system design interviews on YouTube from people who understand why certain decisions are made. Just ask them enough questions.

For example, if a candidate says that using an auto-incrementing primary key as an external identifier for a resource is problematic, ask them for examples of potential attack vectors. On the other hand, if they suggest to use it as an external identifier, ask them to figure out why that can be problematic.

Also, if a candidate suggests `$buzzword` for something that you don’t think warrants it, that might be a bad sign. Ask them why they suggested `$buzzword`.

### Ability to write clean code
I'm sure you've heard some version of [Robert Martin's](https://en.wikipedia.org/wiki/Robert_C._Martin) quote:

> “Indeed, the ratio of time spent reading versus writing is well over 10 to 1. We are constantly reading old code as part of the effort to write new code. ...[Therefore,] making it easy to read makes it easier to write.”

I don't think this is a wrong or particularly controversial opinion. Unfortunately, the interview processes of most, if not all, companies I've interviewed with (which were almost exclusively algo/DS interviews) didn't test the readability of my code. 

I think there a few good ways to see if a candidate can write good "real" code:

1. Ask the candidate for an open-source code sample, if they have one.
2. Give them a take-home or live API design challenge that they can implement, perhaps mocking datastores or whatever other details aren’t reasonable to expect from a coding challenge. 


For assessing the above, you can look for a few things:

- **Implementation of clean abstractions** between application layers via design patterns.
Good naming of variables, functions, classes, etc. Different people have different opinions on this, so I won’t dwell.
- **Good documentation** is a good way to see if people’s technical communication skills are up-to-par. It’s especially important to be able to write good documentation if you’re going to be writing a lot of systems that communicate with each other, and where most of the problems occur at integration points.
- **Unit tests** that catch edge cases and mock things like the filesystem/network cleanly.
- Whatever else you think makes for “clean code”.

### Curiosity 
Asking candidates to tell you about one or two interesting things they’ve learned, or books they’ve read is a nice way to see if they’re just doing what everyone else is doing, or whether they’re genuinely curious and are learning new things.

Bonus personal [example](https://www.usenix.org/legacy/event/lisa07/tech/full_papers/hamilton/hamilton_html/index.html) - an awesome paper about building web services by [James Hamilton](https://mvdirona.com/jrh/work/) I read a little while ago.

### Humility
Designing distributed systems and interfaces is challenging and often involves a lot of mistakes. It’s important to look for people that understand this, and there are a few specifics to look for:

- Engineers that think in a “failure-first” mindset. Rather than designing the “perfect system,” it’s important to assume failure will happen (and more often, as you scale) and apply some stability patterns like load balancing, redundancy, timeouts, retries, and circuit breakers.
- Engineers that are willing to change their minds during the system design interview if you suggest better solutions.
- In terms of interface design, engineers should design APIs that are coherent and easy to understand, but also easy to evolve in case mistakes are made or product requirements change.

### Track record of “above and beyond” achievements at past jobs/projects/etc

This point is directed more towards early-stage startups that, like it or not, need their employees to go above and beyond. 

If the following are true then you shouldn’t expect them to have a wildly different level of impact at this company than they did at their last one.

1. Your company isn’t doing anything that they seem genuinely passionate about (it’s hard to be honest about this).
2. They don’t have a high potential financial upside from the company succeeding.

Ask yourself (genuinely), “if they haven’t gone above and beyond at previous jobs, why would they do it here?” If you can’t find a good answer then you might not want to risk hiring them. 

Note that the answer to the above question doesn't have to be clear cut. Maybe you just have a gut feeling from talking to them that they’ll be an amazing employee, and maybe you should trust your gut.

# Red flags to look for

1. **Jumping to solutions too quickly.** Distributed systems are big and complex, and making a lot of mistakes needlessly is expensive.
2. **Not jumping to solutions quickly enough.** Distributed systems are big and complex, and you will make mistakes along the way, so at some point, decisions have to be made.
Obviously, these last two points are a bit tongue-in-cheek, but this is an extremely important balance to look for in an infra engineer.
3. **“Ivory-tower architects.”** This is a reference from the book Release It! by Michael Nygard (highly recommend). Ivory-tower architects are engineers that try to create (and impose on others) all-encompassing abstractions and standards that are impossibly painful to use in practice. Architecture Astronauts is a similar term for this type of engineer. I find that they’re more common among infrastructure engineers, and so that’s why I’m including this point. This is especially common among API designers who design interfaces for a living, and often want to design the “master interface” to take care of everything. I’ve experienced this firsthand multiple times (and I’ve done it myself!), and it ain’t fun:(
   - A red flag for this is when candidates use a lot of buzzwords and talk about “best practices” with no mention of their downsides or alternatives when suggesting solutions to design problems.
