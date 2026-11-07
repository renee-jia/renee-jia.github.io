---
title: "The Web Is Not a Neutral Environment for Agents"
date: 2026-03-15
research_kind: perspective
categories:
  - Research Blog
  - AI Agents
  - AI Research
tags:
  - browser agents
  - AI safety
  - prompt injection
  - goal fidelity
  - web agents
excerpt: "Browser agents are getting better fast. But the web is full of things that try to steer you, and it works on people every day. Why would agents be any different?"
read_time: "5-10 min read"
layout: distill
last_modified_at: 2026-11-07T21:00:00-08:00
---

Browser agents are getting better fast. OpenAI has Operator and ChatGPT agent. Anthropic has its computer-use models. Open-source tools like Browser Use make it easy for anyone to build an agent that opens websites and gets things done there.

So this is no longer just a fun demo. People are building systems that are meant to really work on the web.

That is what makes one worry stand out to me. The web is full of things that try to steer you. It works on people all day long. I don't see why agents would be any different.

## The web is messy by default

When we talk about agents browsing the web, it's easy to picture a clean setup. The agent has a goal. It reads a few pages, compares what it finds, and decides what to do.

The real web is not like that.

It is full of ads, popups, sponsored results, fake urgency, SEO spam, pages pretending to be official, and text written to keep you on the page or push you toward a button. People already struggle with this. We click the wrong thing, trust the wrong site, or get pulled into a bad flow all the time.

So when an agent moves through the same place, the safe default is not to treat the web as plain information. A lot of the time, it is also influence.

## The problem is bigger than wrong answers

What makes agents different from a plain language model is that they don't just read. They read and then act.

If a chatbot reads a sketchy page, you might get a bad summary. If an agent reads a sketchy page, it might click the wrong button, trust the wrong site, fill in the wrong form, or wander into the wrong flow altogether.

So the question is not only whether the model understands the page. It is whether it can keep its own judgment while the page is working on that judgment.

## This is not only about prompt injection

Prompt injection is part of the story. But the problem is wider than that.

A page does not need to say "ignore previous instructions" to steer an agent. It can do it with a misleading layout, a fake badge of authority, confusing wording, or a string of small nudges that slowly push the agent off course.

That is also how the web works on people. Most of the time there is no single big trick. There is just enough friction, persuasion, or confusion to make the wrong path feel like the natural one.

That is why the real challenge here is not instruction-following. It is goal fidelity. Can the agent stay loyal to the user's goal when everything around it is pulling the other way?

## Why this matters more now

Agents are now being built to act, not just to write. Operator and ChatGPT agent are meant to browse, click, and finish tasks for the user. Anthropic's computer-use work points the same way. And the open-source tools mean many more people are trying it.

So the gap I keep coming back to is this. An agent that looks good in a clean demo and an agent that holds up on the real internet are two very different things.

The real internet is messy, and much of it was built to grab attention or push an action. Once agents spend real time there, how easy they are to steer will matter a lot more than it does today.

## For now

For now, my view is simple. An agent that browses the web is not just looking things up. It is moving through a place that is always trying to change what it does.

So being capable is not enough. It also has to be hard to push around.

To me, that is one of the biggest gaps between an impressive browser-agent demo and something I would actually trust.
