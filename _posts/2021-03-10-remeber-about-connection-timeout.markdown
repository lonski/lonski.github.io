---
layout: post
title: "Remember about connection timeout"
subtitle: ""
date: 2021-03-10
categories: [programming]
---

As a programmer I tend to think optimistically, I focus on how to make something work, not how to break it. Sometimes it results with poorly tested 'bad scenario' paths. 

One thing about I forgot recently (again!) is to set *connection timeout*. Libraries often do not require setting it. Just call `connect` and you are done. Assumption that 'there is probably some default timeout' somewhere is incorrect. Often there is no timeout at all. 

With infinite connection timeout, when something goes wrong your code can just *hang up*, without any exception or anything. It may get worse if you periodically schedule a job, that then hangs up and you may end up with army of zombie jobs eating your yummy resources.
