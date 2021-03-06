---
layout: post
title: "PyConES 2015 - Distributed services"
author: Julio Trigo
date: 2016-02-02 20:40:00 +0100
last_modified_at: 2020-05-08 18:00:00 +0000
permalink: /articles/pycones-2015-distributed-services/
redirect_from: /posts/pycones-2015-distributed-services/
tags:
  - python
  - django
  - boto
  - sqs
youtube_id: LYVfOfR3mp0
youtube_title: "Having it All: Distributed services with Django, Boto, and SQS queues - Julio Trigo"
---

Here you can find all the information about the talk that I gave at [PyConES 2015](http://2015.es.pycon.org) on the 22nd of November, 2015: ***Having it All: Distributed services with Django, Boto, and SQS queues***

Slides: [PyConES 2015 - Distributed Services](https://slides.com/juliotrigo/pycones2015-distributed-services)

<!--more-->

{% include youtubePlayer.html youtube_id=page.youtube_id youtube_title=page.youtube_title %}

Here's the excerpt of the talk:

*How do you let untrained people in your company run sensitive processes on different remote servers? Processes that must run asynchronously and sequentially while accessing different common resources? And how do you do it quickly and make it robust?*

*I will show how we used Django, SQS and Boto to create a distributed and decoupled solution that let users invoke services asynchronously, which is secure, scalable and ensures that processes using common resources ran in sequence.*
