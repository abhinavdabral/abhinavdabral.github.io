---
layout: post
title: My experience with Traefik & Nginx proxy manager
---

I recently started using [Traefik](https://github.com/traefik/traefik) as a reverse proxy for my homelab but before that I was using [Nginx proxy manager](https://github.com/NginxProxyManager/nginx-proxy-manager). Both projects are very capable and this post is just a summarization of my experiences with both the projects.

_Last updated: 2023-08-15_

# Nginx proxy manager

Before moving over to Traefik, I was using nginx proxy manager and it was working perfectly fine for the past 2+ years. There were no major issues with it but there were a few mild inconveniences which encouraged me to try out something different and that's why I moved over to Traefik.

## The Good

- Easy to use, specially for individuals new to self-hosting.
- Entirely configurable from the UI.
- Automatic certificate renewal (using built in Let's Encrypt).
- Very easy to deploy using docker.
- It's a wrapper over `nginx`, which is great.
  - `nginx` itself is a very robust & time-tested web server & reverse proxy
  - `nginx` configurations are well documented

## The Bad

- Not beginner friendly when things break.
  - More on this in next section.
- Not maintained enough with huge backlog of 470 issue labeled `bug` as of writing this.
  - [A CVE was reported](https://github.com/NginxProxyManager/nginx-proxy-manager/issues/2063) in May, 2022
  - [Someone finally patched it](https://github.com/NginxProxyManager/nginx-proxy-manager/pull/2635) on March, 2023
  - This is understandable as this is only maintained as a side-project (in contrast to a project that is actively developed and maintained by a team that's getting paid to do it.)

## The Ugly

- Breaks too easily
  - Host unavailable? Boom
  - Incorrect permissions? Boom
  - Can't even access it's own UI to disable that host anymore
  - You have to go and find whatever config file that host is configured in and disable it manually (or move that file elsewhere temporarily)
- Configuration files
  - There is no easy way to find the configuration file that you're looking for, other than just checking all of them one-by-one.
  - They're named as `1.conf`, `2.conf` and so on.

## Overall

I personally think that Nginx proxy manager will continue to be a very good option for individuals new to self-hosting. It has a few quirks but nothing major as a deal breaker for hobbyists.

It's a small but very capable project which I hope gets more attention and collaborators in future.

# Traefik

Fun fact, Traefik was my first-choice of reverse proxy when I started my self-hosting journey. I seemed daunting and maintaining it was not the most fun part because mostly I was just throwing configuration at it and hoping that it would work, without actually fully understanding what I was doing.

But, that has changed now. Unfortunately, the state of documentation has not improved a whole lot but there are a lot more community resources and forums available now.

## The Good

- Used at enterprize level, which provides a lot of benefits to the project.
  - Actively developed & maintained at a larger scale
  - Less susceptible to lingering bugs or security issues
- Cloud-native solution.
- There are multiple ways to do things.
  - Which is great, only if you know what you're doing.
- Comes with its own dashboard.
- Automatic certificate renewals (using built in Let's Encrypt).
- Auto-discovery feature with docker is great for anyone who wants to use it.
- Dynamic configurations are very handy.

## The Bad

- Steep learning curve
- Errors are not verbose enough for new users.
- v1 vs v2 configuration causes confusions.
- Documentation is not great.
- There are multiple ways to do things.
  - It requires a lot of patience to make sense of things.
  - Copy-pasting often will not work

## The Ugly

- Differences in parameters between different forms configuration.
  - Like in docker labels you're usually writing everything in `lowercase`, but in YAML configuration, we have to use `camelCase`
  - It seems like a minor inconvenience but consider a case where you're trying to troubleshoot something but the solution you find is using a different format. You have to then refer to official documentation and make sure that you're porting that configuration correctly.

## Overall

Traefik is a wonderful solution but it's not ideal for someone new to the dev-ops related tasks, like self-hosting. Based on prior experience it may take you little or a lot of time to make sense of the configurations. Once you get the basic idea of how things are working; everything else will start to make sense as well.

# Conclusion

I wasn't too happy with my Nginx proxy manager setup and that was the main reason I wanted to give Traefik another go (get it? Sorry!). And I'm happy to report that the migration took less time than I anticipated -- just a couple of hours, including the time it took me to re-learn and figure things out all over again. But this time I was able to create a very structured and decoupled setup which makes a lot of more sense to me.

I'll be sharing that post soon.
