---
permalink: "/war-hotel/"
title:  "WAR Hotel"
date:   2016-08-10 16:00:00 -0400
layout: post
categories:
  - WAR Hotel
tags:
  - Chef
  - ruby
  - infrastructure
  - web server
---

# WAR Hotel Part 1 -- Introduction

### Background

What's a WAR Hotel?  It's a big server (VM or bare metal) that can host a whole bunch of WARs. If you don't
know anything about WARs, here's some prerequisite reading:

* [Wikipedia article on WARs](https://en.wikipedia.org/wiki/WAR_(file_format))
* [An introduction to Apache Tomcat](http://tomcat.apache.org/tomcat-9.0-doc/introduction.html)
* [Overview of Pivotal tc Server](http://tcserver.docs.pivotal.io/docs-tcserver/topics/intro-getting-started.html)

WARs have been around a long time and were typically hosted on a server administered by someone who configured
the server directly for the WAR(s) running on the server. It was easy when it was just one running instance of
Tomcat on one server. The administrator was easily able to keep the server up and running, maintaining intimate
knowledge of the server and what was required to host the WAR(s) on the server.

Well, things grew. First it was hosting 10s of WARs on a few servers, then 100s of WARs on scores of servers,
now the demand is 1000s of WARs.  No way we can keep up with this by hand. Platform as a Service (PaaS) solutions
like Pivotal Cloud Foundry or Redhat OpenShift make claims to solve this, but they are walled gardens that lock you
in and have yet to really strategically settle as viable alternatives (come on, admit it, they are still bets).

So to handle the scale problem with a transitional architecture until PaaS settles, there's an interesting approach
we can take.  And why is this approach on this blog? Because it's a great way to learn about infrastructure coding
and Chef! Let's build a WAR hotel using Chef and a couple of other tools that don't require big bucks and lots of
people to maintain.

### Enter Infrastructure Coding

Tools like [Chef](https://chef.io) allow us to solve for this problem of scale.  They were in fact created just for
this purpose.  Don't worry about going off to [Chef.io to learn](https://learn.chef.io), let's learn with a real example here and slowly, deliberately build out the WAR hotel. It'll be rudimentary at first (remember this is designed to do things like homeschool my kids), but when we're done, we'll be doing some complex stuff on a single node, which is something that could ultimately have you [earning over $100K per year](http://venturebeat.com/2015/03/19/30-tech-skills-that-will-get-you-a-110000-plus-salary/).
