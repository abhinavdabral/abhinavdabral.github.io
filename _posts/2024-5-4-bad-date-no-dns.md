---
layout: post
title: Unbound not resolving DNS? Check the server's clock
---

So I've got a Raspberry Pi 4 Model B running OpenWRT. I am also running AdGuard Home and Unbound in Docker containers on the same unit. It was performing as intended, except if I restart the Pi, the DNS would stop responding to queries.

_Last updated: 2024-05-4_

# TL;DR

In my case, it wasn't responding to queries because my system's date & time was not updated. Somehow it was running 2 days behind. So I had to change OpenWRT's NTP servers from domains to IPs, because domains were not getting resolved in the first place.

# Longer version, a troubleshooting tale

## 1. Finding out what is working and what's not

### 1.a Is it DNS?

- I tried to ping `1.1.1.1` and I'm getting replies.
- Then, I tried to ping `google.com` and ... nothing.

I opened my network's DNS setting and set `1.1.1.1` and `1.0.0.1` there.

- Then again, I tried to ping `google.com` and now we're getting replies.

Jeff Geerling has a T-Shirt, which says "It was DNS.". And indeed it was.

Now was it OpenWRT, AdGuard Home, or Unbound?

### 1.b Is AdGuard Home accessible as DNS itself?

I quickly open AdGuard Home's dashboard. Then I tried to resolve a domain using:
```
$ nslookup google.com 192.168.1.1
```
Where `192.168.1.1` is the IP of my Pi 4B running OpenWRT, AdGuard Home and unbound.
Same can be done with `dig` such as `dig google.com @1.1.1.1`.

It couldn't resolve the DNS, obviously, but when I checked AdGuard Home to see if it detected the request, it actually did. I could see the request to `google.com` appear on recent queries.

So that rules out any fault in OpenWRT's DHCP. The DNS port opened from AdGuard home

### 1.c Is AdGuard Home's upstream unreachable?

I opened AdGuard Home and went to DNS settings. In the Upstream DNS, I could see the unbound container's IP address.

I clicked on "Test Upstream" and it shows that it is working.

Which means AdGuard is able to connect to unbound's container as intended.

So this rules out any fault in AdGuard Home's upstream configuration.

### 1.d That leaves us with Unbound

I logged into OpenWRT's shell (and no I wasn't using portainer in this setup).

Started following logs for the unbound container.
`$ docker logs unbound -f` 

```
notice: init module 0: subnetcache
warning: subnetcache: prefetch is set but not working for data originating from the subnet module cache.
notice: init module 1: validator
notice: init module 2: iterator
info: start of service (unbound 1.19.3).
info: generate keytag query _ta-4f66. NULL IN
```

Pretty standard looking logs. No errors here. I tried to resolve domains on my machine and still nothing obvious was getting logged.

I opened the `unbound.conf` and started checking the settings, and I came across `verbosity: 1`. There was a comment on top of it stating that there were 5 levels of it and 1 being the least verbose of them all.

So, I set it to `5` and saved it and exited.

Restarted the docker-containers.

And now I started monitoring the logs again. A lot of stuff was getting logged now.

So I went ahead and tried `nslookup` again. This time we had more details in logs and one particular log line seemed interesting.

```
info: validator: inform_super, sub is com. DS IN
info: super is google.com. A IN
info: verify rrset com. DS IN
debug: verify sig 5613 8
info: verify: signature bad, current time is before inception date expi=20240516050000 incep=20240503040000 now=20240501092417
```

It essentially is telling that the signature is bad because:
- `expi`ry is set to `16th of May 2024`
- `incep`tion is at `3rd of May 2024`
- `now` is `1st of May 2024` 

But it's not `1st of May 2024`

I checked date set on the OpenWRT console by running `date` and it tells me that the system date is running at `1st of May 2024`

After a quick search I landed here at [[Solved] Clock keeps goign horribly out of sync](https://forum.openwrt.org/t/solved-clock-keeps-going-horribly-out-of-sync/133409/3)

- So I tried the command mentioned by Bill there, to set the time from a NTP server.
```
$ ntpd -d -n -q -N -I eth1 -p 162.159.200.123 -p 203.114.74.17
```
- `eth1` here is the WAN device here. 
  - By the way, first IP is for `time.cloudflare.com` (Cloudflare's NTP server)
  - And second one is some Indonesian NTP server

While Bill suggested to add this to `rc.local`, I only wanted to update this one-time just to ensure that it was indeed Date that was the issue.

After executing that command and updating the system clock, I checked date and it updated it, as expected.

Now I tried to do another `nslookup` to resolve the DNS and this time it was getting resolved. Success!

## 2. Conclusion / Solutions

### 2.a Current solution

In my case the problem was time and the solution for me was just to ensure that NTP servers that I was using in my setup were IPs and not domain names.
- I opened the OpenWRT's settings for NTP (**System > System > Time Synchronization** )
- Within NTP server candidates, I removed all the named servers and replaced them with IPs.
  - `162.159.200.123`
  - `203.114.74.17`
- Did a quick restart to ensure that Unbound keeps responding after restart.

And it works! Everything works.


### 2.b Alternate solution

If you have a device that have a hardware RTC (Real-time clock) then that can take care of maintaing the system's time regardless whether your system is turned on or turned off.

There's such a module available for Raspberry Pi 4 as well and other. Raspberry Pi 5 comes with it inbuilt and just needs external battery to make it work.

So one could go with that if you don't want to rely on NTP servers alone.