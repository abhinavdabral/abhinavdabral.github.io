---
layout: post
title: Importance of wildcard certificates in a self-hosted environment
---

Wildcard certificates are convenient for sure, but they're just as equally important as well. In this blog we'll discuss why we should opt for wildcard certificates, specially for our self-hosted setups and how they can help us against certain types of attacks.

_Last updated: 2023-11-1_

# 1. What are wildcard certificates?

In short, whenever we issue certificates for a domain name from a CA (Certificate Authority), we have a choice to:
  - Either get certificate that's valid for specific domain names and sub-domain names. 
  - Or a wildcard certificate, which can be applicable for specific domain names and all its subdomains.

## 1.1 Certificate without wildcard

In this case, the domain names are specified at the time the certificate request is made. These domain names are predefined and do not have any wildcard character. 

### 1.1.1 Example
We can get a single certificate that's only valid for following domain names.
- auth.mydomain.com
- nextcloud.mydomain.com
- mydomain.com
- www.mydomain.com

In case we wish to add a new subdomain, let's say `yolo.mydomain.com`, we'll have to raise a new certificate request with this subdomain added to the list.


## 1.2 Wildcard certificates

For wildcard certificate requests, we specify the domain and subdomain names with wildcard character. These allow us to use the same certificate for as many subdomains as we wish to without having to generate the certificate again (as long as they can match that wildcard).

### 1.2.1 Example

If we request a wildcard certificate that mentions the following - `*.mydomain.com, *.internal.mydomain.com, mydomain.com`, it will be valid for following domains:
  - anything.mydomain.com
  - dashboard.internal.mydomain.com
  - mydomain.com

At any point if we wish to add a new subdomain, let's say `nextcloud.mydomain.com`, we can just do it and use the exact same certificate that we already have, without having to regenerate a new one.

# 2. Why are wildcard certificates important?

Now this post is specifically aimed at small-scale self-hosted setups or small businesses as well. The expectation here is that we don't have dedicated team for security and administration. Therefore, the last thing we need is someone brute-forcing into our Home Assistant and leaving our garage door open, while we're not at home.

## 2.1 Details within a certificate

A little bit of background, without getting into details of [how certificates work](https://www.cloudflare.com/learning/ssl/how-does-ssl-work/).

The part of certificate that gets downloaded to user's browsers is "public", in essence, the details it contains are not considered private. Anyone can see contents of the certificates and all CAs are required to keep this information updated and accessible at all times.

### 2.1.1 Example of wildcard certificate: Wikipedia.org
Try opening [Wikipedia](https://wikipedia.org) and open the certificate. We can see something like this.

![Wikipedia.org certificate details](/images/20240104-wikipedia-certificate-details.png)

As we can see, in case of Wikipedia, they're using same certificate for multiple domains and wildcard subdomains.

### 2.1.2 Example of non-wildcard certificate
Try opening [Amazon India](https://amazon.in) and open the certificate. We can see something like this.

![Amazon.in certificate details](/images/20240104-amazon-certificate-details.png)

## 2.2 Identity search on crt.sh

[crt.sh](https://crt.sh/) is a tool that can be used to do a full-text search on all the information that is indexed by it. All the information is public, so there's no surprize there. 

The results contain all matching domains and identities, along with issuer name and certificate's validity.

As a potential attacker, it's just a matter of finding the right service that is accessible and trying to log in with the default username and password.

# 3. How do we protect our self-hosted setups?

## 3.1 FreeDNS like services

While services like these are convenient, if someone decides to lookup the shared domain name (for example, on `crt.sh`), they'll likely also find the subdomain are logged under it (and using Certificate), including ours. At that point, whatever opens up at the root of that domain is at the mercy of the attacker.

### 3.1.1 In such cases, ensure that
1. SSO is setup for all the exposed services
2. Subdirectories of the services that are hosted at not-so-predictable subdirectories.
   - `yolo.shareddomain.com/nextcloud` <- Predictable
   - `yolo.shareddomain.com/file-stash` <- Un-predictable
3. Don't host anything critical at the root that is publicly exposed. 
4. Don't use default or predictable passwords ANYWHERE.

### 3.2 If we own a domain, use wild-card certificates 

Unless we've a very good reason not to, we should likely be using wildcard certificates to secure our setups.

### 3.2.1 And ALWAYS use certificates when exposing anything to internet

Certificates are necessary to ensure private communication between the user and server. Without that, the traffic can easily be monitored and logged by any of the nodes that it passes through, specially at the ISPs.

# 4. Final thoughts

Self-hosting is a great hobby but it can quickly turn into a nightmare if you get targeted by a particularly curious attacker. While this post focuses on importance of wildcard certificates, it's really important to stay in the loop about all other kind of attack vectors. We must always strive to learn about what these are and how to mitigate them.