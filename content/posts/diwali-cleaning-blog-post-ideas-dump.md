---
title: "Diwali Cleaning: blog post ideas dump"
date: 2023-10-21T13:56:02+05:30
slug: 2023-10-21-diwali-cleaning-blog-post-ideas-dump
type: posts
draft: false
categories: ["meta", "programming"]
tags: ["random", "macos", "terminal", "ux"]
summary: "Diwali cleaning is a seasonal cleaning tradition in India (like Spring cleaning). I'm cleaning up my notes and dumping post ideas that I'm never going to get to."
---

Diwali cleaning is a seasonal cleaning tradition in India (like Spring cleaning). I'm cleaning up my notes and dumping post ideas that I'm never going to get to. Either because I don't have enough expertise or it's just not something that can be stretched into a post.

...

## 1. MacOS default scrollbar visibility

MacOS by default hides scrollbars everywhere, unless you interact with the scrollable element by scrolling on it. This is a good example of prioritizing form over function.

Some issues I've seen with this:

- most obvious issue is that you can't always tell if something is scrollable
- I've seen UIs where devs leave (or not notice) overflows in the UI which result in a big fat scrollbar (which is the right way to do scrollbars) being shown to windows users, making it a terrible experience
  - imagine a dropdown menu with a slight horizontal overflow being hidden in MacOS but showing up for windows users

--

## 2. Timestamps in terminal for executed commands

I've found myself often wonder when exactly did I run a command, how long as it been since I've done it. I'm sure there are prompts out there that solve this already, so it's just a matter of me trying to find it.

Usually goes like this:
- build code
- run terraform apply
- before confirming the run, doubt yourself as to whether you've built with the latest changes, or did you not run build at all
- instead of checking the timestamp on the built file, cancel the run, rebuild and then apply again

--

## 3. AWS odd/confusing metric names

I eventually got used to it and now prefer it this way but it was really odd to me that "maxReceiveCount" on an SQS queue meant the count of messages received by a consumer, not the count of messages received by the queue. 

Always felt that it was odd, as I associated "recieveCount" to be the count of messages received by the queue. The metric is on the queue after all.

Once I got used to it, I feel indifferent to this, however at the time I had made this note, I strongly believed that the labels for "receive" and "sent" metrics should be swapped.

...

That's it for now.