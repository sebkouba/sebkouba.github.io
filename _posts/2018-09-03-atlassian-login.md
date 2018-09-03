---
layout: post
title:  "Atlassian JIRA / Confluence / Hipchat and LoadBalancer Timeouts"
date:   2018-09-03 11:26:43 +0200
categories: atlassian jira confluence haproxy aws
tags:
  - technical
---
### tl;dr
If you need a second login attempt with any of your Atlassian Applications authenticating against Crowd or JIRA reduce your load balancer / reverse proxy timeouts!

### HaProxy and AWS Load Balancer
I've seen two systems where logins from Confluence and HipChat would constantly fail on the first attempt and then succeed on the second.

Error messages were along the lines of:
```
User Directory 'JIRA test Server' is not functional during authentication of 'username'. Skipped.
```
This was for an environment with HaProxy as reverse proxy. Confluence would try to authenticate against Crowd and often, but not always fail on the first attempt.
The workaround in that case was to target the local ip instead of the load balancer.

But then the problem appeared again when using the AWS Classic Loadbalancer. In that case HipChat and Confluence were authenticating against JIRA. The symptoms were the same. 
I coincidentally found that reducing the timeout on the Load Balancer fixed the problem. We went from 3600s, as per Atlassian AWS Cloudformation Template to 60s. Problem solved.

So I went back to the HaProxy system and changed the timeouts there also:
```
We went from:
timeout connect 50s
timeout client 500s
timeout server 500s
To:
timeout connect 10s
timeout client 60s
timeout server 60s
```
Once again, problem solved.
